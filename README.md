
MacOS10.13(High Sierra)に色々なフォーミュラをインストールする、2023年3月

HomeBrew MacOS10.13は通常のインストールではソースからでもビルド出来なくなってきました

完全に無理なら諦めますが、なんとかソースなどの書き換えで対応出来そうです

まず、llvm@12 これは絶対必要で通常インストール出来るフォーミュラです

ただ、Xcode.app(10.1)をインストールし起動しないとllvm@12もインストール出来ません

最初にXcode.appを起動した時に出るライセンス認証ダイアログのOKボタンを押すだけです

brew install llvm@12; その後、

vim /usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # の80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

2023年3月現在、freetype、jpeg-xl、php、mysql などは以下でインストール出来ます

brew install --cc=llvm_clang \<Formula>


nodeやislなどはllvm(llvm@15)が無いとインストール出来ないので書き換えが必要になります

llvm(llvm@15)のインストールですが、いくつか方法があるようでネタ元のリンクを貼っておきます

https://stackoverflow.com/questions/69906053/how-to-install-llvm13-with-homerew-on-macos-high-sierra-10-13-6-got-built-tar

今回はダイレクトに書き換えます、ソースは /tmp にコピーされるので /tmp に移動します

ターミナルをもう１枚開くか、iTerm2なら画面分割で top -u コマンドで動作確認します

brew install --cc=llvm_clang llvm@15

これでソースがダウンロードされ展開され /tmp にコピーされます

topコマンドで bsdtar、ruby、cp と動作確認が出来たらmake前にllvmフォルダーに入って

/tmp/llvm...../llvm...../lldb/source/Host/macosx/objcxx/HostInfoMacOSX.mm の236行目

if (cputype == CPU_TYPE_ARM64 && cpusubtype == CPU_SUBTYPE_ARM64E) {

これを以下に書き換えます

if (cputype == CPU_TYPE_ARM64) {

これでllvm(llvm@15)のインストールが出来ます、その後、

vim /usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # の80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@15/bin/#{Regexp.last_match(1)}"

brew install --cc=llvm_clang isl

islはこれでインストール出来ますが、gccは通常インストールして下さい</br>
(詳しく分かりませんが x86_64、i386 コンパイルで違うようです)

\# 2023年4月末、llvm@16.0.2 がリリースされました、インストール方法は同じです

llvm@16以降はヘッダーをコメントにしないと動きません

vim /usr/local/opt/llvm/include/c++/v1/new; # の355行目

return ::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);

これを以下に書き換えます

return; //::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);

~~llvm 16.0.1 がリリースされました、libtoolライブラリーが古いのでビルドエラーになります~~

~~添付のXcode(12.4)の libtoolに差し替えればインストール出来ますが機能しません~~

~~/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/libtool~~

node のビルドは llvm@15で大丈夫です、node.rbでビルドに llvmが必要なので llvm 16.0.2が無いないなら

brew edit node : 36行目、以下をコメントにして下さい

on_macos do  
&emsp;&emsp;depends_on "llvm" => [:build, :test] if DevelopmentTools.clang_build_version <= 1100  
end</br></br>

nodeはヘッダーのコピーと書き換えが必要になります

システムに触れるのでSystem Integrity Protection(SIP)を無効にして下さい

cd /usr/include/os

sudo cp /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/os/signpost.h .

sudo vim /usr/include/os/signpost.h ; # 280行目

#define os_signpost_event_emit(log, event_id, name, ...) \\  
&emsp;&emsp;        os_signpost_emit_with_type(log, OS_SIGNPOST_EVENT, \\  
&emsp;&emsp;&emsp;                 event_id, name, ##__VA_ARGS__)

これを以下に書き換えます

#define os_signpost_event_emit(log, event_id, name, ...)  
&emsp;&emsp;       //  os_signpost_emit_with_type(log, OS_SIGNPOST_EVENT, \\  
&emsp;&emsp;&emsp;                 event_id, name, ##__VA_ARGS__)

brew install --cc=llvm_clang node

2023年3月、通常インストールと --cc=llvm_clang を使い分ければMacOS10.13でもまだ使えます</br></br>

2023年3月末、ghostscriptは通常インストールや --cc=llvm_clangでもエラーになります

ghostscriptは gccに依存するのでインストールオプションを変え、gccでコンパイルします

brew install --cc=gcc-13 ghostscript</br></br>

2023年3月末、tcl-tkは 8.6バージョン対応のヘッダーを持ち込みインストールに使うようです

10.13は /usr/includeに 8.5対応の tk.hのヘッダーがシンボリックされてる為に

インストールのさいに /usr/includeのヘッダー読み込みが優先されエラーになります

大胆に /use/includeの tk.hを置き換えます

sudo mv /usr/include/tk.h /usr/include/tk_.h

これで、tcl-tkのインストールが出来ます、完了後 tk.hヘッダーを戻します

sudo mv /usr/include/tk_.h /usr/include/tk.h

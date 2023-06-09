
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

\# 2023年6月、llvm@16.0.5 がリリースされました、インストール方法は同じです

llvm@16以降で nodeをビルドする場合はヘッダーをコメントにしないとエラーになります

vim /usr/local/opt/llvm/include/c++/v1/new; # の355行目

return ::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);

これを以下に書き換えます

return; //::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);</br></br>

node のビルドは llvm@15で大丈夫です、node.rbでビルドに llvmが必要なので llvm 16.0.5が無いないなら

brew edit node : 36行目、以下をコメントにして下さい

\# on_macos do  
\# &emsp;&emsp;depends_on "llvm" => [:build, :test] if DevelopmentTools.clang_build_version <= 1100  
\# end</br></br>

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

MacBook Pro Mid 2012 で　npm のアップデートされないので  
usr/local/lib/node_modules/npm を調べると何故か所有者 root になってました  
フォルダー内を調べると所有者、破茶滅茶になってるので chown -R で全て自分に変更しました</br></br>

php はllvm(llvm@16.0.5)でビルドすると　/usr/include/os/signpost.hを読み込みヘッダーでエラーが出ます

/usr/include/os/signpost.h 226行目、251行目を nodoと同じ様に書き換えれば良いのですが面倒です

node用に /usr/include/os/signpost.hを置いてるなら　phpのビルドには llvm@15を使いましょう

2023年3月、通常インストールと --cc=llvm_clang を使い分ければMacOS10.13でもまだ使えます

rust はビルドの順番があるようです --cc=llvm_clangでいきなりビルド指定するとエラーになります</br></br>

2023年3月末、ghostscriptは通常インストールや --cc=llvm_clangでもエラーになります

ghostscriptは gccに依存するのでインストールオプションを変え、gccでコンパイルします

brew install --cc=gcc-13 ghostscript</br></br>

~~2023年3月末、tcl-tkは 8.6バージョン対応のヘッダーを持ち込みインストールに使うようです~~

~~10.13は /usr/includeに 8.5対応の tk.hのヘッダーがシンボリックされてる為に~~

~~インストールのさいに /usr/includeのヘッダー読み込みが優先されエラーになります~~

~~大胆に /use/includeの tk.hを置き換えます~~

~~sudo mv /usr/include/tk.h /usr/include/tk_.h~~

~~これで、tcl-tkのインストールが出来ます、完了後 tk.hヘッダーを戻します~~

~~sudo mv /usr/include/tk_.h /usr/include/tk.h~~</br></br>

2023年5月 tar のバージョンが古い為に  doxygen のファイル展開すら出来なくなりました

brew install gnu-tar

/usr/bin/tar は /usr/bin/bsdtar のシンボリックです、元があるので消えても復元できます

sudo mv /usr/bin/tar /usr/bin/tar_buck</br>
sudo ln -s /usr/local/bin/gtar /usr/bin/tar

目的のフォーミュラーを展開しインストール出来れば tar を元に戻しましょう

sudo rm /usr/bin/tar</br>
sudo mv /usr/bin/tar_buck /usr/bin/tar

/usr/local/bin のパスが先に読み込まれ、CUIのみ使用なら以下で使えます

ln -s /usr/local/bin/gtar /usr/local/bin/tar

2023年6月末　openldap はメイク後、既存のファイルを参照してリンクするようです

アップグレードする場合は置換エラーになるので既存ファイルを削除してアップグレードして下さい

rm /usr/local/etc/openldap/slapd.conf /usr/local/etc/openldap/slapd.ldif

10.13 環境の問題では無い様です　,  https://github.com/orgs/Homebrew/discussions/4586</br></br>

subversion は llvm がインストールされていればメイクに clang と clang-15 を併用します

10.13 純正 clang のアーキテクチャは i386 , clang-15 のアーキテクチャは x86_64 になります

clang と clang-15 はアーキテクチャが違うので llvm があるとエラーになります

subversion をインストールする場合は llvm のリンクを解除してインストールして下さい

brew unlink llvm@15  
brew install subversion

subversion のインストールが終われば llvm のリンクを戻して大丈夫です

brew link llvm@15

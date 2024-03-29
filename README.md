
MacOS10.13(High Sierra)に色々なフォーミュラをインストールする、2023年3月

HomeBrew MacOS10.13は通常のインストールではソースからでもビルド出来なくなってきました

完全に無理なら諦めますが、なんとかソースなどの書き換えで対応出来そうです

HomeBrew 4.0.0以降の場合はシェルに以下の環境変数を設定して下さい
```
echo 'export HOMEBREW_NO_INSTALL_FROM_API=1' >> ~/$(echo .${SHELL##*/}rc)
```
まず、llvm@12 これは必要で通常インストール出来るフォーミュラです

ただ、Xcode.app(10.1)をインストールし起動しないとllvm@12もインストール出来ません

https://developer.apple.com/download/more/ # ここからダウンロード

最初にXcode.appを起動した時に出るライセンス認証ダイアログのOKボタンを押すだけです

brew install llvm@12 ; その後、

/usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # 80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

2023年12月 php、isl、mysql、jpeg-xl などは以下でインストール出来ます

brew install --cc=llvm_clang \<Formula>

--cc オプションは指定したフォーミュラにのみ有効で依存するフォーミュラには動作しません  
なので依存するフォーミュラに --cc オプションが必要な場合は個別にインストールします</br></br>

2024年3月 node(21.7.1)は llvm@15が無いとインストール出来ないので書き換えが必要になります

llvm@15のインストールですが、いくつか方法があるようでネタ元のリンクを貼っておきます

https://stackoverflow.com/questions/69906053/how-to-install-llvm13-with-homerew-on-macos-high-sierra-10-13-6-got-built-tar

今回はダイレクトに書き換えます、ソースは /tmp にコピーされるので /tmp に移動します

ターミナルをもう１枚開くか、iTerm2なら画面分割で top -u コマンドで動作確認します

brew install --cc=llvm_clang llvm@15

これでソースがダウンロードされ展開され /tmp にコピーされます

topコマンドで bsdtar、ruby、cp と動作確認が出来たらmake前にllvmフォルダーに入って

/tmp/llvmA15...../llvm-project-15.0.7.src/lldb/source/Host/macosx/objcxx/HostInfoMacOSX.mm # 236行目

if (cputype == CPU_TYPE_ARM64 && cpusubtype == CPU_SUBTYPE_ARM64E) {

これを以下に書き換えます

if (cputype == CPU_TYPE_ARM64) {

これで llvm(llvm@15)のインストールが出来ます、その後、

/usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # 80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@15/bin/#{Regexp.last_match(1)}"</br></br>

node(21.7.1) のビルドは llvm@15で大丈夫です、node.rbでビルドに llvmが指定されてるので llvm(17.0.6_1)が無いないなら

brew edit node ; # 36行目、以下をコメントにして下さい

\# on_macos do  
\# &emsp;&emsp;depends_on "llvm" => [:build, :test] if DevelopmentTools.clang_build_version <= 1100  
\# end</br></br>

nodeはヘッダーのコピーと書き換えが必要になります

システムに触れるので System Integrity Protection(SIP)を無効にして下さい

https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection

sudo cp /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/os/signpost.h /usr/include/os/

sudo vim /usr/include/os/signpost.h ; # 280行目

#define os_signpost_event_emit(log, event_id, name, ...) \\  
&emsp;&emsp;        os_signpost_emit_with_type(log, OS_SIGNPOST_EVENT, \\  
&emsp;&emsp;&emsp;                 event_id, name, ##\_\_VA_ARGS__)

これを以下に書き換えます

#define os_signpost_event_emit(log, event_id, name, ...)  
&emsp;&emsp;       //  os_signpost_emit_with_type(log, OS_SIGNPOST_EVENT, \\  
&emsp;&emsp;&emsp;                 event_id, name, ##\_\_VA_ARGS__)

brew install --cc=llvm_clang node<br/><br/>

python 関係で ModuleNotFoundError: No module named 'packaging' エラー等が出る場合

mkdir ~/.pip  
vim ~/.pip/pip.conf # 以下２行を書く

[global]  
break-system-packages = true

cd /usr/local/Cellar/python@3.12/3.12.0/bin  
./python3.12 -m pip install --upgrade pip  
./python3.12 -m pip install 'packaging'<br/><br/>

2024年1月 llvm(17.0.6_1) がリリースされました、インストール方法は llvm@15 と同じです

llvm(17.0.6_1) 関連のインストールは依存関係が少しややこしいです

vim は ruby に依存し ruby のビルドに rust が必要になります

rust は llvm に依存し llvm のビルドに ninja が必要になります<br/><br/>

llvm(17.0.6_1) のビルドは llvm@15 と同じです、ただ私の環境では何故か  
iMac(2013)OS10.13 はビルド出来るのですが iBookPro(2012)OS10.13 ではエラーになります

~/Library/Logs/Homebrew/llvm/02.cmakeを確認すると引数が足りないエラーになっています

CMake Error at /tmp/llvm......../llvm-project-17.0.6.src/compiler-rt/cmake/Modules/CompilerRTUtils.cmake:371 (string):
8515   string sub-command REPLACE requires at least four arguments.

ビルドに成功してる iMacで足りない値を表示すると　x86_64-apple-darwin17.7.0 でした  
cc --version で返ってくる値　Target: x86_64-apple-darwin17.7.0 になります  
/tmp/llvm......../llvm-project-17.0.6.src/compiler-rt/cmake/Modules/CompilerRTUtils.cmake # 370行目に追加

set(COMPILER_RT_DEFAULT_TARGET_TRIPLE "x86_64-apple-darwin17.7.0")</br></br>

llvm@16以降で nodeをビルドする場合はヘッダーをコメントにしないとエラーになります

/usr/local/opt/llvm/include/c++/v1/new; # の355行目

return ::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);

これを以下に書き換えます

return; //::aligned_alloc(__alignment, __size > __rounded_size ? __size : __rounded_size);

php を llvm@16 以降でビルドするとエラーになります、特別な理由がなければ llvm@15でビルドしましょう</br></br>

2023年10月 libheif はビルド依存する pkg-config が gdk-pixbuf のパスを読み込めずエラーになります

mv /usr/local/Cellar/gdk-pixbuf/2.42.10_1 /usr/local/Cellar/gdk-pixbuf/2.42.10

libheif のインストールが終われば元に戻しましょう

mv /usr/local/Cellar/gdk-pixbuf/2.42.10 /usr/local/Cellar/gdk-pixbuf/2.42.10_1</br></br>

gccは通常インストールして下さい、ただllvm@15をインストールしていると

llvm@15のライブラリにリンクしようとするのでエラーになります

brew unlink llvm@15 # これでllvm@15のライブラリは消えます

gccのインストールが終われば元に戻しましょう brew link llvm@15</br></br>

2023年3月末、ghostscriptは通常インストールや --cc=llvm_clangでもエラーになります

ghostscriptは gccに依存するのでインストールオプションを変え、gccでコンパイルします

brew install --cc=gcc-13 ghostscript</br></br>

shared-mime-info も --cc=gcc-13 オプションを使って下さい

brew install --cc=gcc-13 shared-mime-info<br/><br/>

subversion は llvm がインストールされていればメイクに clang と clang-15 を併用します

10.13 純正 clang のアーキテクチャは i386 , clang-15 のアーキテクチャは x86_64 になります

clang と clang-15 はアーキテクチャが違うので llvm があるとエラーになります

subversion をインストールする場合は llvm のリンクを解除してインストールして下さい

brew unlink llvm@15 ; brew install subversion

subversion のインストールが終われば llvm のリンクを戻して大丈夫です

brew link llvm@15

2024年2月 graphvizでエラーになります  
強制インストールすれば動作しますがバグが残りそうです

brew install --debug graphviz

BuildError: Failed executing: make  
1\. raise  
2\. ignore  
3\. backtrace  
4\. irb  
5\. shell  
Choose an action: 2

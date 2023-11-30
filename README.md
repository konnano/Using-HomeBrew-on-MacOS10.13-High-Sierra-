
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

2023年11月 freetype、jpeg-xl、php、isl、mysql などは以下でインストール出来ます

brew install --cc=llvm_clang \<Formula>

--cc オプションは指定したフォーミュラにのみ有効で依存するフォーミュラには動作しません  
なので依存するフォーミュラに --cc オプションが必要な場合は個別にインストールします</br></br>

2023年11月 node(21.2.0)は llvm@15が無いとインストール出来ないので書き換えが必要になります

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

node(21.2.0) のビルドは llvm@15で大丈夫です、node.rbでビルドに llvmが指定されてるので llvm(17.0.6)が無いないなら

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

OS10.13 では python@3.12 で機能しない部分があります

python-packaging は python@3.12 でエラーになります

brew edit python-packaging

22行目、以下をコメントにして下さい

\# depends_on "python@3.12" => [:build, :test]

sphinx-doc も python@3.12 でエラーが出る場合は

brew edit sphinx-doc # 33行目

depends_on "python@3.12"

これを以下に書き換えます

depends_on "python@3.11"

python 関係で ModuleNotFoundError: No module named 'packaging' エラー等が出る場合

mkdir ~/.pip  
vim ~/.pip/pip.conf # 以下２行を書く

[global]  
break-system-packages = true

cd /usr/local/Cellar/python@3.12/3.12.0/bin  
./python3.12 -m pip install --upgrade pip  
./python3.12 -m pip install 'packaging'<br/><br/>

2023年11月 llvm(17.0.6) がリリースされました、インストール方法は llvm@15 と同じです

iMac(2013)OS10.13 ではビルド出来るのですが iBookPro(2012)OS10.13 ではエラーになります

~/Library/Logs/Homebrew/llvm/02.cmakeを確認すると何故か引数が足りないとエラーになっています

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

rust は llvm が必要になります configure オプションで llvm のパスを読み込んでくれるので

llvm があれば通常インストールして下さい<br/><br/>

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

2023年7月 gnu-tar 1.35 はバグがある様で　configure オプションで

--disable-nls で言語サポートを無効にしないとインストールエラーになります

brew edit gnu-tar ; # 37行目に追加

args << if OS.mac?  
&emsp;&emsp;"--program-prefix=g"  
&emsp;&emsp;"--disable-nls"  
else

dpkg は gnu-tar に依存するのですが gtar のパスを探そうとしてエラーになります

ln -s /usr/local/Cellar/gnu-tar/1.35/bin/tar /usr/local/opt/gnu-tar/bin/gtar</br></br>

ronn などの gem を使うフォーミュラは、新しい gem を使います

brew install ruby

brew edit ronn # 37行目

system "gem", "build", "ronn.gemspec"  
system "gem", "install", "ronn-#{version}.gem"

これを以下に書き換え

system "/usr/local/opt/ruby/bin/gem", "build", "ronn.gemspec"  
system "/usr/local/opt/ruby/bin/gem", "install", "ronn-#{version}.gem"

brew install ronn<br/><br/>

subversion は llvm がインストールされていればメイクに clang と clang-15 を併用します

10.13 純正 clang のアーキテクチャは i386 , clang-15 のアーキテクチャは x86_64 になります

clang と clang-15 はアーキテクチャが違うので llvm があるとエラーになります

subversion をインストールする場合は llvm のリンクを解除してインストールして下さい

brew unlink llvm@15 ; brew install subversion

subversion のインストールが終われば llvm のリンクを戻して大丈夫です

brew link llvm@15

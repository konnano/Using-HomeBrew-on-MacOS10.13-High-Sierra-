
MacOS10.13(High Sierra)に色々なフォーミュラをインストールする、2023年3月

HomeBrew MacOS10.13は通常のインストールではソースからでもビルド出来なくなってきました

完全に無理なら諦めますが、なんとかソースなどの書き換えで対応出来そうです

HomeBrew 4.0.0以降の場合はシェルに以下の環境変数を設定して下さい
```
export HOMEBREW_NO_INSTALL_FROM_API=1
```
まず、llvm@12 これは絶対必要で通常インストール出来るフォーミュラです

ただ、Xcode.app(10.1)をインストールし起動しないとllvm@12もインストール出来ません

https://developer.apple.com/download/more/ # ここからダウンロード

最初にXcode.appを起動した時に出るライセンス認証ダイアログのOKボタンを押すだけです

brew install llvm@12; その後、

vim /usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # の80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

2023年9月 freetype、jpeg-xl、php、isl などは以下でインストール出来ます

brew install --cc=llvm_clang \<Formula></br></br>

2023年9月 nodeはllvm(llvm@15)が無いとインストール出来ないので書き換えが必要になります

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

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@15/bin/#{Regexp.last_match(1)}"</br></br>

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

rust はビルドの順番があるようです --cc=llvm_clangでいきなりビルド指定するとエラーになります

通常インストールすれば clang, clang-15 と順番にビルドしてくれます</br></br>

gccは通常インストールして下さい、ただllvm@15をインストールしていると

llvm@15のライブラリにリンクしようとするのでエラーになります

brew unlink llvm@15 # これでllvm@15のライブラリは消えます

gccのインストールが終われば元に戻しましょう brew link llvm@15

2023年3月末、ghostscriptは通常インストールや --cc=llvm_clangでもエラーになります

ghostscriptは gccに依存するのでインストールオプションを変え、gccでコンパイルします

brew install --cc=gcc-13 ghostscript

2023年9月 HomeBrewにバグがある為、--cc=gcc-13 が使えないので書き換えて下さい  

vim /usr/local/Homebrew/Library/Homebrew/extend/ENV/shared.rb # 351行目 

when GNU_GCC_REGEXP  
&emsp;&emsp;other # <= 書き換え => other.to_sym  
else

~~2023年5月 tar のバージョンが古い為に  doxygen のファイル展開すら出来なくなりました~~

~~brew install gnu-tar~~

~~/usr/bin/tar は /usr/bin/bsdtar のシンボリックです、元があるので消えても復元できます~~

~~sudo mv /usr/bin/tar /usr/bin/tar_buck</br>~~
~~sudo ln -s /usr/local/bin/gtar /usr/bin/tar~~

~~目的のフォーミュラーを展開しインストール出来れば tar を元に戻しましょう~~

~~sudo rm /usr/bin/tar</br>~~
~~sudo mv /usr/bin/tar_buck /usr/bin/tar~~

2023年7月 gnu-tar 1.35 はバグがある様で　configure オプションで  
--disable-nls で言語サポートを無効にしないとインストールエラーになります  
brew edit gnu-tar # 37行目

args << if OS.mac?  
&emsp;&emsp;"--program-prefix=g"  
&emsp;&emsp;"--disable-nls"  
else

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

2023年9月 mysql 8.1.0のビルドでエラーになります、brew install --cc=llvm_clang mysql # llvm@12以上

mysqlメーリングリストの諸先輩にご指導頂きインストール出来ました

mysql、HomeBrewどちらのバグか分かりませんが、なぜかopenssl@1.1ヘッダーを読み込み  
openssl@3にリンクする為にエラーになります

mv /usr/local/opt/openssl@1.1/include /usr/local/opt/openssl@1.1/include_buck

これでopenssl@3のヘッダーを読みに行ってくれます

mysql 8.1.0のインストールが終わればopenssl@1.1/includeの場所戻しときましょう


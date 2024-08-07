
MacOS10.13(High Sierra)に色々なフォーミュラをインストールする、2023年3月

HomeBrew MacOS10.13は通常のインストールではソースからでもビルド出来なくなってきました

完全に無理なら諦めますが、なんとかソースなどの書き換えで対応出来そうです

HomeBrew 4.0.0以降はシェルに以下の設定をして下さい
```
echo 'export HOMEBREW_NO_INSTALL_FROM_API=1' >> ~/$(echo .${SHELL##*/}rc)
source  ~/$(echo .${SHELL##*/}rc)
brew tap homebrew/core
```
まず、llvm@12 これは必要で通常インストール出来るフォーミュラです

ただ、Xcode.app(10.1)をインストールし起動しないとllvm@12もインストール出来ません

https://developer.apple.com/download/more/ # ここからダウンロード

最初にXcode.appを起動した時に出るライセンス認証ダイアログのOKボタンを押すだけです

``brew install llvm@12`` ; その後、

/usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # 80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@12/bin/#{Regexp.last_match(1)}"

2023年12月 php、jpeg-xl などは以下でインストール出来ます

``brew install --cc=llvm_clang`` \<Formula>

--cc オプションは指定したフォーミュラにのみ有効で依存するフォーミュラには動作しません  
なので依存するフォーミュラに --cc オプションが必要な場合は個別にインストールします</br></br>

islのビルドに gcc@11が必要なのですが gcc@11が islに依存するので gccの通常インストールが出来ません

フォーミュラを書き換え(削除)するので保存しておきます

cp /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/g/<foo>gcc</foo>@11.rb ~/<foo>gcc<foo>@11.rb

brew edit gcc@11 # islを無効にします

\# depends_on "isl" # 30行目をコメントに

--with-isl=#{Formula["isl"].opt_prefix} # 75行目を削除

--disable-bootstrap # 78行目に追加、とりあえずビルド

``brew install gcc@11``

``brew install --cc=gcc-11 isl``

mv  ~/<foo>gcc</foo>@11.rb /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/g/<foo>gcc</foo>@11.rb

``brew install --cc=gcc-11 gcc``

llvmでビルドできた islを reinstallしてみたら --cc=gcc-11オプションでエラーになります  
llvmでビルドできた islでは --cc=llvm_calngを使わないといけない様です  
なぜこうなるのか、原因がわかりません</br></br>

2024年5月 node(22.2.0)は llvm@15が無いとインストール出来ないので書き換えが必要になります

llvm@15のインストールですが、いくつか方法があるようでネタ元のリンクを貼っておきます

https://stackoverflow.com/questions/69906053/how-to-install-llvm13-with-homerew-on-macos-high-sierra-10-13-6-got-built-tar

今回はダイレクトに書き換えます、ソースは /tmp にコピーされるので /tmp に移動します

ターミナルをもう１枚開くか、iTerm2なら画面分割で top -u コマンドで動作確認します

``brew install --cc=llvm_clang llvm@15``

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

node(22.2.0) のビルドは llvm@15を使ってください、node.rbでビルドに llvmが指定されてるので

brew edit node ; # 36行目、以下をコメントにして下さい

\# on_macos do  
\# &emsp;&emsp;depends_on "llvm" => [:build, :test] if DevelopmentTools.clang_build_version <= 1100  
\# end</br></br>

node(22.2.0) はヘッダーのコピーと書き換えが必要になります

[依存する c-aresもヘッダーの書き換えが必要になります](#10)

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

``brew install --cc=llvm_clang node``<br/><br/>

python 関係で ModuleNotFoundError: No module named 'packaging' エラー等が出る場合  
エラー表示が python@3.11、python@3.12 どちらか確認して下さい

mkdir ~/.pip  
vim ~/.pip/pip.conf # 以下２行を書く

[global]  
break-system-packages = true

cd /usr/local/Cellar/python@3.12/3.12.3/bin \# or python@3.11/3.11.9/bin  
./python3.12 -m pip install --upgrade pip \# or ./python@3.11   
./python3.12 -m pip install 'packaging' \# or ./python@3.11<br/><br/>

2024年6月 llvm@15のライブラリが欠落して、node(22.3.0)以降にアップデート出来ません  
<details><summary>templateの書き換えで何とかなるかもしれませんが、私にはわかりません</summary>
  
```
Undefined symbols for architecture x86_64:
  "std::__1::__fs::filesystem::path::__filename() const", referenced from:
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4path6appendB6v15007INS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEEEENS_9enable_ifIXsr13__is_pathableIT_EE5valueERS2_E4typeERKSB_ in libnode.a(node_task_runner.o)
      __ZNKSt3__14__fs10filesystem4path8filenameB6v15007Ev in libnode.a(node_errors.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::path::__root_name() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path9root_nameB6v15007Ev in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::path::__parent_path() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path11parent_pathB6v15007Ev in libnode.a(node_task_runner.o)
      __ZNKSt3__14__fs10filesystem4path11parent_pathB6v15007Ev in libnode.a(node_modules.o)
Undefined symbols for architecture x86_64:
  "std::__1::__fs::filesystem::path::__filename() const", referenced from:
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4path6appendB6v15007INS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEEEENS_9enable_ifIXsr13__is_pathableIT_EE5valueERS2_E4typeERKSB_ in libnode.a(node_task_runner.o)
      __ZNKSt3__14__fs10filesystem4path8filenameB6v15007Ev in libnode.a(node_errors.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::path::__root_directory() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path9root_pathB6v15007Ev in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::path::__root_name() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path9root_nameB6v15007Ev in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::path::__compare(std::__1::basic_string_view<char, std::__1::char_traits<char> >) const", referenced from:
      node::modules::BindingData::TraverseParent(node::Realm*, std::__1::__fs::filesystem::path const&) in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::path::__parent_path() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path11parent_pathB6v15007Ev in libnode.a(node_task_runner.o)
      __ZNKSt3__14__fs10filesystem4path11parent_pathB6v15007Ev in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::__equivalent(std::__1::__fs::filesystem::path const&, std::__1::__fs::filesystem::path const&, std::__1::error_code*)", referenced from:
      node::task_runner::FindPackageJson(std::__1::__fs::filesystem::path const&) in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::path::__root_directory() const", referenced from:
      __ZNKSt3__14__fs10filesystem4path9root_pathB6v15007Ev in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_task_runner.o)
      __ZNSt3__14__fs10filesystem4pathdVB6v15007ERKS2_ in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::__current_path(std::__1::error_code*)", referenced from:
      node::task_runner::RunTask(std::__1::shared_ptr<node::InitializationResultImpl>, std::__1::basic_string_view<char, std::__1::char_traits<char> >, std::__1::vector<std::__1::basic_string_view<char, std::__1::char_traits<char> >, std::__1::allocator<std::__1::basic_string_view<char, std::__1::char_traits<char> > > > const&) in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::path::__compare(std::__1::basic_string_view<char, std::__1::char_traits<char> >) const", referenced from:
      node::modules::BindingData::TraverseParent(node::Realm*, std::__1::__fs::filesystem::path const&) in libnode.a(node_modules.o)
  "std::__1::__fs::filesystem::path::replace_extension(std::__1::__fs::filesystem::path const&)", referenced from:
      node::ReportFatalException(node::Environment*, v8::Local<v8::Value>, v8::Local<v8::Message>, node::EnhanceFatalException) in libnode.a(node_errors.o)
  "std::__1::__fs::filesystem::__equivalent(std::__1::__fs::filesystem::path const&, std::__1::__fs::filesystem::path const&, std::__1::error_code*)", referenced from:
      node::task_runner::FindPackageJson(std::__1::__fs::filesystem::path const&) in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::__status(std::__1::__fs::filesystem::path const&, std::__1::error_code*)", referenced from:
      node::task_runner::FindPackageJson(std::__1::__fs::filesystem::path const&) in libnode.a(node_task_runner.o)
  "std::__1::__fs::filesystem::__current_path(std::__1::error_code*)", referenced from:
      node::task_runner::RunTask(std::__1::shared_ptr<node::InitializationResultImpl>, std::__1::basic_string_view<char, std::__1::char_traits<char> >, std::__1::vector<std::__1::basic_string_view<char, std::__1::char_traits<char> >, std::__1::allocator<std::__1::basic_string_view<char, std::__1::char_traits<char> > > > const&) in libnode.a(node_task_runner.o)
ld: symbol(s) not found for architecture x86_64
  "std::__1::__fs::filesystem::path::replace_extension(std::__1::__fs::filesystem::path const&)", referenced from:
      node::ReportFatalException(node::Environment*, v8::Local<v8::Value>, v8::Local<v8::Message>, node::EnhanceFatalException) in libnode.a(node_errors.o)
clang-15: error: linker command failed with exit code 1 (use -v to see invocation)
```
        
</details>
        
残念ながら node(22.2.0)が最終バージョンになりました、以下が依存関係のバージョンです  
```
node (22.2.0)
├── brotli (1.1.0)
├── c-ares (1.30.0)
├── icu4c (74.2)
├── libnghttp2 (1.61.0)
├── libuv (1.48.0)
├── openssl@3 (3.3.1)
│   └── ca-certificates (2024-03-11)
├── python@3.12 (3.12.4)
│   ├── mpdecimal (4.0.0)
│   ├── openssl@3 (3.3.1)
│   │   └── ca-certificates (2024-03-11)
│   ├── sqlite (3.46.0)
│   │   └── readline (8.2.10)
│   ├── xz (5.4.6)
│   ├── libffi (3.4.6)
│   ├── ca-certificates (2024-03-11)
│   ├── readline (8.2.10)
│   └── gettext (0.22.5)
├── ca-certificates (2024-03-11)
├── mpdecimal (4.0.0)
├── readline (8.2.10)
├── sqlite (3.46.0)
│   └── readline (8.2.10)
├── xz (5.4.6)
└── libffi (3.4.6)
```
2024年8月 node(22.2.0) のボトルを置いておきます、不具合があれば教えて下さい

`brew install node--22.2.0.high_sierra.bottle.tar.gz`</br></br>

2024年6月 llvm(18.1.8) がリリースされました、インストール方法は llvm@15 と同じです

llvm(18.1.8) 関連のインストールは依存関係が少しややこしいです

vim は ruby に依存し ruby のビルドに rust が必要になります

rust は llvm に依存し llvm のビルドに ninja が必要になります<br/><br/>

llvm(18.1.8) のビルドは llvm@15 と同じです、ただ私の環境では何故か  
iMac(2013)OS10.13 はビルド出来るのですが iBookPro(2012)OS10.13 ではエラーになります

~/Library/Logs/Homebrew/llvm/02.cmakeを確認すると引数が足りないエラーになっています

CMake Error at /tmp/llvm......../llvm-project-18.1.8.src/compiler-rt/cmake/Modules/CompilerRTUtils.cmake:371 (string):
8515   string sub-command REPLACE requires at least four arguments.

ビルドに成功してる iMacで足りない値を表示すると　x86_64-apple-darwin17.7.0 でした  
cc --version で返ってくる値　Target: x86_64-apple-darwin17.7.0 になります  
/tmp/llvm......../llvm-project-18.1.8.src/compiler-rt/cmake/Modules/CompilerRTUtils.cmake # 370行目に追加

set(COMPILER_RT_DEFAULT_TARGET_TRIPLE "x86_64-apple-darwin17.7.0")</br></br>

2024年8月 mysql(9.0.1) がリリースされました

mysql(8.3.0) から (9.0.1) へダイレクトにバージョンアップが出来ません  
mysql(8.4.0) をインストールしてから (9.0.1) にアップグレードして下さい  
アップグレードの方法は幾つかあります https://github.com/orgs/Homebrew/discussions/5539  
mysql@8.4 を使う場合は mysql@8.4 の依存関係の書き換えとインストール方法の変更が必要になります

色々試してみましたが結果的にデーターベースのバックアップを取って  
mysql のアンインストール、/usr/local/var/mysql の削除してから  
mysql(9.0.1) をダイレクトにインストールするのが一番簡単でした

アップグレードするなら mysql フォーミュラをコピーして保存して下さい

cp /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/m/mysql.rb ~/  

`brew edit mysql`

配布先 https://cdn.mysql.com/Downloads/MySQL-8.4/mysql-8.4.0.tar.gz

mysql フォーミュラのダウンロード先と sha256 を mysql(8.4.0) に書き換えて下さい

<details><summary> 私のフォーミュラを載せておきます</summary>
  
```
class Mysql < Formula
  desc "Open source relational database management system"
  homepage "https://dev.mysql.com/doc/refman/9.0/en/"
  url "https://github.com/konnano/Using-HomeBrew-on-MacOS10.13-High-Sierra-/releases/download/mysql.8.4.0/mysql-8.4.0.tar.gz"
  sha256 "47a5433fcdd639db836b99e1b5459c2b813cbdad23ff2b5dd4ad27f792ba918e"
  license "GPL-2.0-only" => { with: "Universal-FOSS-exception-1.0" }

  livecheck do
    url "https://dev.mysql.com/downloads/mysql/?tpl=files&os=src"
    regex(/href=.*?mysql[._-](?:boost[._-])?v?(\d+(?:\.\d+)+)\.t/i)
  end

  bottle do
    sha256 arm64_sonoma:   "809ff45d66016c434d3eca504cd4f9b251101134bcf48e53c332de6ee12c1007"
    sha256 arm64_ventura:  "73e6042f446a8c97788509ee5abd64c7fa41c93c1e151a7d28fb7a48a49b9551"
    sha256 arm64_monterey: "15e1db8916ac543e67ccadb30a7c89a9febef12006162df29d302930e2e11270"
    sha256 sonoma:         "7a3c3908482b71648c894d27ea1e173d20a4afbbf4f02f94da31c161cb1603fe"
    sha256 ventura:        "655afb9130f8e0b86c471683a5b3ece67fbed24004c6e9712b7c8e9f9dd47b1a"
    sha256 monterey:       "101522798c8bbb18bbec25a80016bc23212bde4d611e655f59fa77e531244ec4"
    sha256 x86_64_linux:   "60ea8b0e209b2e640dd675b158ba8f1f8c39dfe6ded013c0e164eef2bfe689f8"
  end

  depends_on "bison" => :build
  depends_on "cmake" => :build
  depends_on "pkg-config" => :build
  depends_on "icu4c"
  depends_on "libfido2"
  depends_on "lz4"
  depends_on "openssl@3"
  depends_on "protobuf@21" # percona-xtrabackup dependency conflict
  depends_on "zlib" # Zlib 1.2.13+
  depends_on "zstd"

  uses_from_macos "curl"
  uses_from_macos "cyrus-sasl"
  uses_from_macos "libedit"

  on_macos do
    depends_on "llvm" if DevelopmentTools.clang_build_version <= 1400
  end

  on_linux do
    depends_on "patchelf" => :build
    depends_on "libtirpc"
  end

  conflicts_with "mariadb", "percona-server",
    because: "mysql, mariadb, and percona install the same binaries"

  fails_with :clang do
    build 1400
    cause "Requires C++20"
  end

  fails_with :gcc do
    version "9"
    cause "Requires C++20"
  end

  def datadir
    var/"mysql"
  end

  def install
    if OS.linux?
      # Fix libmysqlgcs.a(gcs_logging.cc.o): relocation R_X86_64_32
      # against `_ZN17Gcs_debug_options12m_debug_noneB5cxx11E' can not be used when making
      # a shared object; recompile with -fPIC
      ENV.append_to_cflags "-fPIC"

      # Disable ABI checking
      inreplace "cmake/abi_check.cmake", "RUN_ABI_CHECK 1", "RUN_ABI_CHECK 0"
    elsif DevelopmentTools.clang_build_version <= 1400
      ENV.llvm_clang
      # Work around failure mixing newer `llvm` headers with older Xcode's libc++:
      # Undefined symbols for architecture arm64:
      #   "std::exception_ptr::__from_native_exception_pointer(void*)", referenced from:
      #       std::exception_ptr std::make_exception_ptr[abi:ne180100]<std::runtime_error>(std::runtime_error) ...
      ENV.prepend_path "HOMEBREW_LIBRARY_PATHS", Formula["llvm"].opt_lib/"c++"
    end

    # -DINSTALL_* are relative to `CMAKE_INSTALL_PREFIX` (`prefix`)
    args = %W[
      -DCOMPILATION_COMMENT=Homebrew
      -DINSTALL_DOCDIR=share/doc/#{name}
      -DINSTALL_INCLUDEDIR=include/mysql
      -DINSTALL_INFODIR=share/info
      -DINSTALL_MANDIR=share/man
      -DINSTALL_MYSQLSHAREDIR=share/mysql
      -DINSTALL_PLUGINDIR=lib/plugin
      -DMYSQL_DATADIR=#{datadir}
      -DSYSCONFDIR=#{etc}
      -DWITH_SYSTEM_LIBS=ON
      -DWITH_BOOST=boost
      -DWITH_EDITLINE=system
      -DWITH_FIDO=system
      -DWITH_ICU=system
      -DWITH_LIBEVENT=system
      -DWITH_LZ4=system
      -DWITH_PROTOBUF=system
      -DWITH_SSL=system
      -DWITH_ZLIB=system
      -DWITH_ZSTD=system
      -DWITH_UNIT_TESTS=OFF
      -DWITH_INNODB_MEMCACHED=ON
    ]

    system "cmake", "-S", ".", "-B", "build", *args, *std_cmake_args
    system "cmake", "--build", "build"
    system "cmake", "--install", "build"

    (prefix/"mysql-test").cd do
      system "./mysql-test-run.pl", "status", "--vardir=#{Dir.mktmpdir}"
    end

    # Remove the tests directory
    rm_r(prefix/"mysql-test")

    # Fix up the control script and link into bin.
    inreplace "#{prefix}/support-files/mysql.server",
              /^(PATH=".*)(")/,
              "\\1:#{HOMEBREW_PREFIX}/bin\\2"
    bin.install_symlink prefix/"support-files/mysql.server"

    # Install my.cnf that binds to 127.0.0.1 by default
    (buildpath/"my.cnf").write <<~EOS
      # Default Homebrew MySQL server config
      [mysqld]
      # Only allow connections from localhost
      bind-address = 127.0.0.1
      mysqlx-bind-address = 127.0.0.1
    EOS
    etc.install "my.cnf"
  end

  def post_install
    # Make sure the var/mysql directory exists
    (var/"mysql").mkpath

    # Don't initialize database, it clashes when testing other MySQL-like implementations.
    return if ENV["HOMEBREW_GITHUB_ACTIONS"]

    unless (datadir/"mysql/general_log.CSM").exist?
      ENV["TMPDIR"] = nil
      system bin/"mysqld", "--initialize-insecure", "--user=#{ENV["USER"]}",
        "--basedir=#{prefix}", "--datadir=#{datadir}", "--tmpdir=/tmp"
    end
  end

  def caveats
    s = <<~EOS
      We've installed your MySQL database without a root password. To secure it run:
          mysql_secure_installation

      MySQL is configured to only allow connections from localhost by default

      To connect run:
          mysql -u root
    EOS
    if (my_cnf = ["/etc/my.cnf", "/etc/mysql/my.cnf"].find { |x| File.exist? x })
      s += <<~EOS

        A "#{my_cnf}" from another install may interfere with a Homebrew-built
        server starting up correctly.
      EOS
    end
    s
  end

  service do
    run [opt_bin/"mysqld_safe", "--datadir=#{var}/mysql"]
    keep_alive true
    working_dir var/"mysql"
  end

  test do
    (testpath/"mysql").mkpath
    (testpath/"tmp").mkpath
    system bin/"mysqld", "--no-defaults", "--initialize-insecure", "--user=#{ENV["USER"]}",
      "--basedir=#{prefix}", "--datadir=#{testpath}/mysql", "--tmpdir=#{testpath}/tmp"
    port = free_port
    fork do
      system bin/"mysqld", "--no-defaults", "--user=#{ENV["USER"]}",
        "--datadir=#{testpath}/mysql", "--port=#{port}", "--tmpdir=#{testpath}/tmp"
    end
    sleep 5
    assert_match "information_schema",
      shell_output("#{bin}/mysql --port=#{port} --user=root --password= --execute='show databases;'")
    system bin/"mysqladmin", "--port=#{port}", "--user=root", "--password=", "shutdown"
  end
end
```
        
</details>

`brew unlink boost`  
`mysql.server stop`

最新の llvm((18.1.8) が必要になります

/usr/local/Homebrew/Library/Homebrew/shims/super/cc ; # 80行目

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm@15/bin/#{Regexp.last_match(1)}"

これを以下に書き換えます

"#{ENV["HOMEBREW_PREFIX"]}/opt/llvm/bin/#{Regexp.last_match(1)}"

`brew install --cc=llvm_clang mysql` # mysql(8.4.0)  
`mysql.server start`  
`mysql.server stop`

cp ~/mysql.rb /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/m/

`brew install --cc=llvm_clang mysql` # mysql(9.0.1)</br></br>

2024年5月 libheifはビルド依存する pkg-configが Homebrewのgdk-pixbufを読み込みエラーになります

mv /usr/local/Cellar/gdk-pixbuf/2.42.12 /usr/local/Cellar/gdk-pixbuf/2.42.10

``brew install libheif``

libheif のインストールが終われば元に戻しましょう

mv /usr/local/Cellar/gdk-pixbuf/2.42.10 /usr/local/Cellar/gdk-pixbuf/2.42.12</br></br>

2023年3月末、ghostscriptは通常インストールや --cc=llvm_clangでもエラーになります

ghostscriptは gccに依存するのでインストールオプションを変え、gccでコンパイルします

``brew install --cc=gcc-11 ghostscript``</br></br>

shared-mime-info も --cc=gcc-11 オプションを使って下さい

``brew install --cc=gcc-11 shared-mime-info``<br/><br/>

subversion は llvm がインストールされていればメイクに clang と clang-15 を併用します

10.13 純正 clang のアーキテクチャは i386 , clang-15 のアーキテクチャは x86_64 になります

clang と clang-15 はアーキテクチャが違うので llvm があるとエラーになります

subversion をインストールする場合は llvm のリンクを解除してインストールして下さい

brew unlink llvm@15 ; ``brew install subversion``

subversion のインストールが終われば llvm のリンクを戻して大丈夫です

brew link llvm@15

2024年5月　openexr をリインストールしたらエラーになりました

~/Library/Logs/Homebrew/openexr/01.cmake を見てみると

libdeflate , clang-format がないエラーです、依存関係が更新されてません

brew install libdeflate

brew reinstall openexr

clang-format はなくても大丈夫です</br></br>

2024年5月 <a id=10>c-ares</a> はヘッダーで定義されてないようでエラーになります

c-ares は node に必要なので SIP を無効にして書き換えます

sudo vim /usr/include/dispatch/dispatch.h # 38行目

\#if !defined(HAVE_UNISTD_H) || HAVE_UNISTD_H

HAVE_UNISTD_H 定義を無効にして unistd.hを読み込ませます

\#if !defined(HAVE_UNISTD_H) // || HAVE_UNISTD_H

brew install c-ares 

~/Library/Logs/Homebrew/c-ares/02.cmake を確認すると

has no symbols と警告が出ながらビルドは成功します

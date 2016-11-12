### 1.1 安装 Ruby

许多 Linux 发行版已经预装了 Ruby，macOS 也预装了，可以在命令行提示符下输入 `ruby -v`

#### 1.1.1 二进制发行版

Ruby 的二进制发行版很容易安装，这是为特定操作系统环境而预先编译的。不足之处是你只能用它提供的东西；它可能版本比较低；可能不包含你需要的可选的软件包

对基于 RPM 的 Linux 系统，可以到 `http://www.rpmfind.net` 去搜索一个合适的 Ruby RPM 包。输入 `Ruby` 作为搜索关键字，并从列出的版本号、体系结构和发行版中选择。

对 `Debian` 这样基于 `dpkg` 的 Linux 系统，可以使用 `apt-get` 系统去查找和安装 Ruby，也可以使用 `apt-cache` 命令去搜索 Ruby 包：`apt-cache search ruby iterpreter` 或 `apt-get install ruby1.8`。

在 `Microsoft Windows` 上，可以去 `http://rubyinstaller.rubyforge.org` 上可以找到 Ruby One-Click 安装程序的 主页。

#### 1.1.2 从源码编译 Ruby

因为 Ruby 是一个开放源代码的项目，可以下载 Ruby 解释器的源代码，并在自己的系统上编译。这种方式给予了对安装选项的更多控制，并且能保持安装是最新的。不便之处在于需要管理编译和安装过程。

第一件要做的事是下载源代码，从 `http://www.ruby-lang.org` 上下载源代码有 3 种选择：

* `Tarball` 格式的稳定版（stable release），`Tarball` 是一个归档文件
* 稳定版快照（stable snapshot），这是每天晚上创建的基于 Ruby 稳定版开发分支的最新源代码的 tarball 文件。稳定分支（stable branch）是用于产品发布的代码，一般来说比较可靠。但是由于 snapshot 是每天产生一个，所以新的特性可能没有经过全面的测试。
* Nightly 开发板快照（nightly development snapshot），这也是每天晚上创建的 tarball。这些代码是最新主流代码，因为这是从主开发分支中获得的，这种版本有可能会出现无法预料的问题

下载了 tarball 归档文件后，还需要把它解压缩还原为原来的格式，可以用 `tar` 命令做这项工作：`tar xsf snapshot.tar.gz`

这会安装 Ruby 源代码树（source tree）到 ruby/子目录中。在那个目录中可以找到一个名为 README 的文件，它详细解释了安装过程。总的来说，在基于 POSIX 的系统上编译 Ruby，使用的 4 个命令与编译大多数开放源代码程序所使用的命令相同：`./configure`、`make`、`make test` 和 `make install`。在别的环境（包括 Windows）编译 Ruby，需要使用一个像 cygwin 这样的 POSIX 模拟环境，或者使用自带的编译器一一参照发行版的 win 32 子目录中的 `README.win32` 

#### 1.1.3 本书的源代码

在于 `http://pragmaticprogrammer.com/titles/ruby/code` 上以供下载

### 1.2 运行 Ruby


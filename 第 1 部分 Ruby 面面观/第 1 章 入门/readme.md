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

Ruby 开发者使用 CVS（Concurrent Version System）作为版本控制系统，可以执行下面的命令，以匿名用户将文件从其归档（archive）中检出：

```shell
cvs -z4 -d :pserver:anaoymous@cvs.ruby-lang.org:/src login
cvs -z4 -d :pserver:anaoymous@cvs.ruby-lang.org:/src checkout ruby
```

这个命令将会导出源代码开发树的主分支，比如想导出 Ruby 1.8 分支，可以在 `checkout` 后面加入 `-r ruby_1_8`

#### 1.2.1 交互式 Ruby

可以直接在 `shell` 下执行 `ruby` 来交互式运行，但推荐使用 `irb` 来交互执行 Ruby。irb 是一个 Ruby Shell，包括命令行历史（command-line history）功能、行编辑（line-editing）功能和作业控制（job control）。

可以使用 irb 来运行已经存在于文件中的代码示例，比如：

```shell
irb > load "code/rdoc/fib_example.rb"
irb > Fibonacci.upto(20)
```

#### 1.2.2 Ruby 程序

可以用 Unix 的 `shebang` 符号作为程序文件的第一行：

```ruby
#! /usr/local/bin/ruby -w
```

如果系统支持的话，还可以使用下面这句，系统将会搜索 Ruby 路径并执行之，避免硬性指定 Ruby 路径

```ruby
#! /usr/bin/env ruby
```

如果将此源代码文件设置为可执行，Unix 就可将其作为程序来运行

```shell
./myprog.rb
```

### 1.3 Ruby 文档：RDoc 和 ri

许多库由内部一个称为 RDoc 的系统来进行文档化

如果使用 RDoc 来文档化一个源文件，那么其中的文档可以被提取出来，并转换成 HTML 或者 ri 格式

ri 是一种本地命令行工具，用来阅读 RDoc 格式的 Ruby 文档。

输入 ri 类名，就可以得到该类的文档。比如：`ri GC`

以方法名字作为参数，可以获得该方法的信息：`ri enable`
如果传给 ri 的方法名称存在于多个类或模块当中，ri 会列出所有的可能。在方法名字前加上类名字和点号重新调用：`ri GC.start`

`--format` 选项，可以告诉 ri 如何显示修饰性的文本符号，比如 `--format ansi`

```shell
export RI="--format ansi --width 70"
```
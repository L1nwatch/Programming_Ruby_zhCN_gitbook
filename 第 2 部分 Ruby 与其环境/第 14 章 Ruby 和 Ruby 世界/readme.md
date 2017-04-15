### 14.1 命令行参数

Ruby 命令行由 3 部分组成：传递给 Ruby 解释器的选项，要运行的程序名称（可选）和程序的一组参数（可选）。

```shell
ruby [options] [--] [programfile] [arguments]
```

Ruby 本身的选项截止到命令行中首个以连字符（-）开始的单词、或者特别标志——两个连字符。

如果是命令行上没有指定文件名，或者指定的文件名是单个连字符（-），Ruby 会从标准输入读入程序源码：

```shell
ruby -w - "Hello World"
```

#### 14.1.1 命令行选项

略

#### 14.1.12 ARGV

程序文件后面出现的任何命令行参数在 Ruby 程序中可用，它们保存在全局数组 ARGV 中。比如：

```ruby
ARGV.each {|arg| p arg}
```

注意 `ARGV[0]` 是程序的第一个参数而不是程序名称。当前程序的名称位于全局变量 `$0` 中。注意 ARGV 中所有的值都是字符串。

如果你的程序试图从标准输入（或使用特别文件 ARGF）中读取，ARGV 中的程序参数会被当做文件名称，然后 Ruby 会从这些文件中读取。如果程序把参数和文件名称混在一起，那就确保从这些文件读取之前已经清空了 ARGV 数组中的非文件名参数。

### 14.2 程序终止

`Kernel#exit` 方法会终止程序，返回一个状态值给操作系统。但是与有些语言不一样，exit 没有立即终止程序。`Kernel#exit` 首先抛出可以捕获的 `SystemExit` 异常，然后执行若干个清理动作，其中包括任何已注册的 `at_exit` 方法和对象的终止方法（`finalizers`）。

### 14.3 环境变量

可以使用预定义的变量 ENV 来访问操作系统环境变量。它响应与 Hash 相同的方法（ENV 实际上不是散列表，但是可以使用 `#ENV#to_hash` 把它转换成 Hash。

```ruby
ENV["SHELL"]	# ->	"/bin/sh"
ENV["HOME"]		# ->	"/Users/dave"
ENV["USER"]		# ->	"dave"
ENV.keys.size	# ->	32
ENV.keys[0, 7]	# ->	["MANPATH", ...]
```

某些环境变量的值在 Ruby 第一次运行时被读取。

#### 14.3.1 写入环境变量

Ruby 程序可能写入 ENV 对象。在大多数操作系统上这会改变相应环境变量的值。当然，这种变化仅限于作出这种改变的进程，以及随后被它创建的那些子进程。

Ruby 使用的环境变量

| 变量名称               | 描述                                       |
| ------------------ | ---------------------------------------- |
| `DLN_LIBRARY_PATH` | 动态装入模块的查找路径                              |
| HOME               | 指向用户主目录。在文件名和目录名中展开 ~ 时使用它               |
| LOGDIR             | 如果没有设置 `$HOME`，它是用户主目录的 fallback 指针。只被 `Dir.chdir` 使用 |
| `OPENSSL_CONF`     | 指定 OpenSSL 配置文件的位置                       |
| RUBYLIB            | Ruby 程序的附加查找路径（`$SAFE` 必须为 0）            |
| `RUBYLIB_PREFIX`   | （只限 Windows）通过将前缀添加到 RUBYLIB 的每个组成部分，来改编（mangle）RUBYLIB 查找路径 |
| RUBYOPT            | Ruby 的附加命令行选项；在解析完实际的命令行选项之后被检查（`$SAFE` 必须为 0） |
| RUBYPATH           | 使用 `-S` 选项时 Ruby 程序的查找路径（默认为 PATH）       |
| RUBYSHELL          | 在 Windows 下创建执行进程时要用到的 shell；如果没有设置，则会检查 SHELL 或 COMSPEC 环境变量 |
| `RUBY_TCL_DLL`     | 覆盖（override）TCL 共享库或 DLL 的默认名称           |
| `RUBY_TK_DLL`      | 覆盖 Tk 共享库或 DLL 的默认名称。使用它或者 `RUBY_TCL_DLL` 时都必须要设置这两个选项 |

### 14.4 从何处查找它的模块

使用 `require` 或 `load` 把程序库模块装入到程序中。有些模块是 Ruby 自带的，有些是从 Ruby 应用归档（Ruby Application Archive）安装的，有些是自己开发的，如何进行查找？

为特定机器编译 Ruby 时，它预定义了一组标准目录去保存 Ruby 程序库。可以使用类似下面命令行得到答案：

```ruby
% ruby -e "puts $:"
```

`site_ruby` 目录是为了保存已经添加的那些模块和扩展。

如果程序运行在安全级别 0 上，可以把环境变量 RUBYLIB 设置成一个包含一个或多个查找目录的列表。如果程序没有 `setuid`，可以使用命令行参数 `-I` 来做同样的事情。

Ruby 变量 `$:` 是一个目录数组，用来查找已装入的文件。这个变量被初始化为标准目录表，加上用 `RUBYLIB` 和 `-I` 选项指定的所有附加目录。程序运行过程中随时可以将附加目录添加到这个数组中。

### 14.5 编译环境

为特定的体系结构编译 Ruby 时，所有用来编译它的相关设置（包括用来编译 Ruby 的机器的体系结构，编译器选项和源代码目录等等）都被写入到库文件 `rbconfig.rb` 中的 `Config` 模块。Ruby 安装之后，任何 Ruby 程序可以使用这个模块得到编译 Ruby 的细节。

```ruby
require "rbconfig"
include Config
CONFIG["HOST"]		# ->	"pwoer..."
CONFIG["libdir"]	# ->	"/Users/..."
```

扩展库可以使用这个配置文件在给定的体系结构上正确地编译和链接。
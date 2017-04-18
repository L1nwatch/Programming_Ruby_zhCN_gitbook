RubyGems 是一个库和程序的标准化打包以及安装框架，它使定位、安装、升级和卸载 Ruby 包变得容易。它给使用者和开发者提供了 4 种主要功能：

*   一种标准化的包格式
*   使用一个中央仓库来存储这种格式的包
*   安装和管理同时安装的多个版本的相同库
*   最终用户工具以查询、安装、卸载以及别的方式来操作这些包

在 RubyGems 的世界里，开发人员把它们的程序和库包裹（bundle）到一个 gems 文件中。这些文件遵循标准化的格式，同时 RubyGems 系统提供命令行工具去操作这些 gem 文件，这个工具被恰当地命名为 gem。

### 17.1 安装 RubyGems

测试 RubyGems 是否已安装成功：

```shel
gem help
```

### 17.2 安装程序 Gems

Rake 程序享有第一个以 gem 方式发布的应用的殊荣。它是一个手边常用的好工具，类似于 Make 和 Ant 的一个构建（build）工具。实际上，甚至可以使用 Rake 去构建 gems

```shell
% gem install -r rake	
% rake --version
```

RubyGems 下载了 Rake 库和命令行程序 rake。

`-r` 选项告诉 gem 远程操作。许多 RubyGems 操作可以在本地或远程执行。比如，可以使用 query 命令去显示所有远程可用的 gems 文件或显示已经安装在本地的 gems 列表。`-l` 指定在本地执行。

可以使用 RubyGems 的版本需求操作符（version requirement operator）去指定选择版本的条件：

```shell
gem install -r rake -v"< 0.4.3"
```

在安装的时候也可以添加 `-t` 选项到 RubyGems 的 `install` 命令，这会让 RubyGems 运行这个 `gem` 的测试套件。如果测试失败了，安装程序会提示保留或丢弃这个 gem。

```shell
gem install SomePoorlyTestedProgram -t
```

***

##### 版本操作符

>   说明
>
>   Gem::Specification 的 `require_gem` 方法和 `add_dependency` 属性都接受一个指定版本依赖的参数。RubyGems 版本依赖采用操作符 `major.minor.patch_level` 形式。下面列出的表包含所有可能的版本操作符

| 操作符  | 描述                                       |
| ---- | ---------------------------------------- |
| =    | 精确版本匹配。Major（主）、minor（次）和 patch（补丁）级别必须相同 |
| !=   | 任何不是指定版本的版本                              |
| >    | 任何比指定版本高的版本（甚至在 patch 级别）                |
| <    | 任何比指定版本低的版本                              |
| \>=  | 任何比指定版本高或相等的版本                           |
| \<=  | 任何比指定版本低或相等的版本                           |
| -\>  | "Boxed"（装箱式）版本操作符。版本必须高于或等于指定版本，同时应该低于指定版本在其次版本号加 1 后得到的结果。这是为了避免不同次版本之间 API 的不兼容性 |

### 17.3 安装和使用 Gem 库

```shell
gem query -rn Blue
```

query 命令使用 `-n` 选项在中央 gem 仓库中查找任何名字匹配正则表达式 `/Blue/` 的 gem 文件。

#### 17.3.1 生成 API 文档

在添加 `--rdoc` 选项到 `install` 命令后，RubyGems 会为安装的 gem 生成 RDoc 文档。

```shell
gem install -r BlueCloth --rdoc
```

生成这些 HTML 文档后，可以进入 RubyGems 的文档目录，直接浏览这些文档。每个 gem 文档存储在一个集中受保护的位置，这是 RubyGems 所特有的。查找文档最可靠的办法是执行 gem 命令，询问 RubyGems 主目录放在哪里。

```shell
gem environment gemdir
```

RubyGems 把生成的文档存储到这个目录的 `doc/` 子目录中。

如果发现自己经常使用这个路径，可以创建快捷方式。

```shell
gemdoc=`gem environment gemdir`/doc
ls $gemdoc
open $gemdoc/BlueCloth-0.0.4/rdoc/index.html
```

为了节省时间，可以在登录 shell 的 profile 或 rc 文件里声明 `$gemdoc`

第二种，阅读 gems RDoc 文档的方法是使用 RubyGems 自带的 `gem_server` 实用工具。

```shell
gem_server
```

`gem_server` 启动一个 Web 服务器，运行在启动它的电脑上。默认情况下，它运行在端口 8808 上，从默认的 RubyGems 安装目录为那些 gems 以及它们的文档提供服务。端口号和 gem 目录可以分别通过使用命令行的 `-p` 和 `-d` 选项去重新设置。

#### 17.3.2 开始编码

通过使用 RubyGems 后，就可以利用它的打包和版本支持。

```ruby
require "rubygems"
require_gem "BlueCloth", ">=0.0.4"
doc = BlueCloth::new <<MARKUP
This is some example [text][1]. Just learning to use [BlueCloth][1].
Just a sample test.
[1]： http://ruby-lang.org
MARKUP
puts doc.to-html
```

第二行把 BlueCloth gem 添加到 Ruby 的 `$LOAD_PATH` 中，同时使用 `require` 装入 `gem ` 作者指定要自动装入的所有库。

每个 gem 被认为是一组资源。它可能包含一个或一百个库文件。在老式的非 RubyGems 库中，所有这些文件会被拷贝到 Ruby 程序库树中的某个共享位置，这个位置包括在 Ruby 预定义的装载路径中。

RubyGems 在自包含的目录树中保存每个版本的 gem。gems 并没有被存储到标准的 Ruby 库目录中。其结果是，RubyGems 需要一些技巧才能让你得到那些文件。它是通过将 gem 的目录树添加到 Ruby 的装载路径来实现的。

#### 17.3.3 依赖于 RubyGems

我们至今编写的这些代码都依赖于已安装的 RubyGems 包。然而，RubyGems 还不是标准 Ruby 发行版的一部分，所以使用你的软件的人可能没有把 RubyGems 安装到它们的电脑里。此时如果发行代码有 `require rubygems` 将会出错。

>   幕后的代码
>
>   当调用 `require_gem` 方法时，会发生什么？
>
>   第一，gems 库修改 `$LOAD_PATH` 包括已经添加到 gemspec 中的 `require_paths` 的任何目录。
>
>   第二，它对 gemspec 的 autorequires 属性中指定的任何文件调用 Ruby 的 require 方法。正是这个 `$LOAD_PATH` 的修改行为运行 RubyGems 管理相同库的多个已安装版本。

至少可以使用两种技术去规避这个问题。首先，可以把 RubyGems 特有的代码封装到代码块中，并使用 Ruby 的异常处理去 rescue 这个在 require 时没有找到 RubyGems 而引发的 LoadError。

```ruby
begin
  require "rubygems"
  require_gem "BlueCloth", ">= 0.0.4"
rescue LoadError
  require "bluecloth"
end
```

在 RubyGems 0.8.0 中，请求 `rubygems.rb` 会安装一个重载版本的 Ruby require 方法。

重载的 require 方法几乎可以让程序避免使用任何 RubyGems 特有的代码。一个例外是，在请求这些通过 gem 安装的库之前，RubyGems 库必须先被装载。

可以使用 `-r` 开关调用 Ruby 解释器来避免依赖 RubyGems

```shell
ruby -rubygems myprogram.rb
```

这会导致解释器装入 RubyGems 框架，从而安装了 RubyGems 重载版本的 require 方法。为了在给定系统上确保每次调用 Ruby 解释器时 RubyGems 会被装入，可以设置 RUBYOPT 环境变量：

```shell
% export RUBYOPT=rubygems
```

这样当你运行 Ruby 解释器时无须显式装载 RubyGems 框架，那些以 gem 方式安装的库对需要它们的那些应用是可用的。

使用重载的 require 方法最大的缺点是，失去了管理同一库的多个已安装版本的能力。如果需要特定版本的库，最好使用先前描述的 LoadError 方式。

### 17.4 创建自己的 Gems

#### 17.4.1 包的布局

创建 gem 的首要任务是在一个有意义的目录结构中组织代码。在创建典型的 tar 或 zip 档案时所使用的一些规则，同样适用于对软件包的组织。

*   把所有的 Ruby 源文件放在 `lib/` 子目录中
*   包含一个 `lib/yourproject.rb` 的文件，它会执行必要的 `require` 命令去装入项目的大部分功能。在 RubyGems 的 autorequire 特性出现之前，这使得别人更方便地使用库
*   总是包括 README 文件，其中包含项目概要、作者联系信息以及入门知识的获取。这个文件使用 RDoc 格式，这样可以把它添加到 gem 安装时生成的文档中。README 文件应当包含版权和许可证
*   测试用例应当放在 `test/` 目录中。很多开发者把库的单元测试当做一个用法指南来使用。
*   任何可执行的脚本应当放在 `bin/` 子目录中
*   Ruby 扩展源代码应当放在 `ext/` 中
*   如果 gem 包含非常多的文档，把这些文档保存在它们自己的 `docs/` 子目录中是好的做法

#### 17.4.2 Gem 规范

gem 创建核心：gem 规范或 gemspec。

gemspec 是 Ruby 或 YAML 形式的元数据集，它们提供了关于 gem 的关键信息。gemspec 作为构建 gem 流程的输入。可以使用几种不同的机制来创建 gem，但它们在概念上都是相同的。

```ruby
require "rubygems"
SPEC = Gem::Specification.new do |s|
  s.name		= "MomLog"
  s.version		= "1.0.0"
  s.author		= "Jo Programmer"
  s.email		= "jo@joshost.com"
  s.homepage	= "http://www.joshost.com/MomLog"
  s.platform	= Gem::Platform::RUBY
  s.summary		= "An online Diary for families"
  candidates	= Dir.glob("{bin, docs, lib, test}/**/*")
  s.files 		= candidates.delete_if do |item|
    				item.include?("CVS") || item.include?("rdoc")
				  end
  s.require_path= "lib"
  s.autorequire	= "momlog"
  s.test_file	= "test/ts_momlog.rb"
  s.has_rdoc	= true
  s.extra_rdoc_files = ["README"]
  s.add_dependency("BlueCloth", ">=0.0.4")
end
```

gem 的元数据保存在 `Gem::Specification` 类的对象中。gemspec 可以用 YAML 或 Ruby 代码来表达。

gem 的概述是对软件包的简短描述，运行 `gem query` 时会显示它。

`files` 属性是一个文件数组，编译 gem 时会包括这些文件。

#### 17.4.3 运行时魔术

属性 `require_path` 和 `autorequire` 让你指定当 `require_gem` 装入 gem 时，会被添加到 `$LOAD_PATH` 中的目录，以及通过 `require` 被自动装入的所有文件。

RubyGems 也提供了 `require_paths`，即复数版本。它接收一个数组，允许指定多个将要包含在 `$LOAD_PATH` 中的目录。

#### 17.4.4 添加测试和文档

`test_file` 属性保存 gem 中的单个相对路径文件，这个文件应当作为 `Test::Unit` 测试套件（test suite）被装入。（可以使用复数形式的 `test_files` 去引用一个包含测试的文件数组）

`has_rdoc` 属性表明你已经在代码中添加了 RDoc 注释。在完全没有注释的代码上运行 RDoc 是可行的，只要提供一个其接口的可浏览视图，但是这并不推荐。

RDoc 格式无论以源文件或者其渲染后的形式来看，可读性都非常好，这使得 Rdoc 格式称为 README 文件的一个极好的选择。默认情况下，RDOC 命令只会运行在源代码文件上。`extra_rdoc_file` 属性是 gem 中非源代码文件的一个数组，表明你希望当生成 RDoc 文档时包括这些文件。

#### 17.4.5 添加依赖

`add_dependency` 方法的参数等同于 `require_gem` 的参数。

生成了 gem 后，当试图在干净的系统上安装它时，会看到类似下面的消息：

```shell
% gem install pkg/MomLog1.0.0.gem
```

因为正在从文件中执行本地安装，RubyGems 不会试图解决这个依赖。

可以在一个 gemspec 中多次调用 `add_dependency` 方法，来支持你需要它去解决的那些依赖关系。

#### 17.4.6 Ruby 扩展 Gems

许多 Ruby 库是作为本地扩展方式（native extension）来创建的。作为一个 gem，有两种方式可以去打包和分发这种类型的库。可以用源码格式分发 gem，同时让安装程序在安装时编译代码。另外，可以预编译这个扩展，为每个要支持的独立平台分发一个 gem。

RubyGems 会为源码 gems 提供附加的 `Gem::Specification` 属性，称为 extensions。这个属性是一个 Ruby 文件的数组，它们会生成 Makefiles。创建其中一个程序的最典型方式是使用 Ruby 的 mkmf 库。

必须在规范的 files 列表中包含那些源文件，这样它们会包含在分发的 gem 包中。

当安装源码 gem 时，RubyGems 运行其中的每个 extensions 程序，并执行进而产生的 Makefile。

RubyGems 没有能力检测源码 gems 可能有的系统库依赖。

分发源码的 gem 显然需要这个 gem 的使用者有一套开发工具。至少他们需要某种类型的 make 程序和编译器。尤其对 Windows 用户来说，他们可能没有这些工具。可以通过分发预编译的 gems 规避这种不足。

创建预编译的 gems 很简单，只要把已编译的共享库文件（share object，在 Windows 上是 DLL）添加到 gemspec 的 files 列表中，同时确信这些文件在 gem 的 `require_path` 属性的某个路径中。与纯 Ruby gems 一样，`require_gem` 命令会修改 Ruby 的 `$LOAD_PATH`，共享库会通过 require 来访问。

既然这些 gems 是平台特有的，也可以使用 platform 属性指定 gem 的目标平台。`Gem::Specification` 类为 Windows、Intel Linux、macOS 和纯 Ruby 平台定义了相应常量。对于那些没有包含在这个表中的平台，可以使用 `RUBY_PLATFORM` 变量的值。这个属性现在纯粹只是用来提供信息的，但是使用它是一个应当有的良好习惯。将来的 RubyGems 发行会根据安装程序正在运行的平台，使用 platform 属性去智能地选择已预编译的 gems。

#### 17.4.7 编译 Gem 文件

所创建的 gemspec 是一个可执行的 Ruby 程序，调用它会生成一个 gem 文件：

```shell
% ruby momlog.gemspec
```

另外，可以使用 `gem build` 命令生成 `gem` 文件：

```shell
% gem build momlog.gemspec
```

现在有了 gem 文件，就可以像其他包一样来分发它。

可以通过调用下面的命令来安装 gem（假设已经安装了 RubyGems）：

```shell
% gem install MomLog-0.5.0.gem
```

如果想要把 gem 发行到 Ruby 社区，可以使用 RubyForge。RubyForge 是一个管理开源项目的 Web 站点。它也存储中央 gem 仓库。任何使用 RubyForge 文件发行功能发布的 gem 文件，每天会有多次被自动选取并添加到中央 gem 仓库中。

#### 17.4.8 用 Rake 编译

可以使用 Rake 来编译 gems。Rake 使用一个称为 Rakefile 的命令行文件去控制编译过程。它定义了（Ruby 语法）一组规则和任务。Make 中以规则驱动的概念和 Ruby 能力的结合，创建了一个自动构建和发行的环境。

>   任务是 Rakefile 中主要的工作单元。任务有一个名称（通常以符号或字符串方式给出），一个前提条件的列表（更多的符号或字符串）和一个动作列表（以 block 的方式给出）

一般情况下，你可以使用 Rake 內建的 task 方法在 Rakefile 中定义自己命名的任务。

Rake 提供了特别的 TaskLib，它被称为 GemPackage Task，帮助把 gem 创建整合到你的自动化构建和发行阶段的剩余部分。

为了在 Rakefile 中使用 GemPackageTask，如前面那样创建 gemspec，但是这次是把它放在 Rakefile 中。然后把这个规范传送给 GemPackageTask。

```ruby
略
```

注意必须请求 rubygems 包到 Rakefile 中。`FileList` 比 `Dir.glob` 更智能，它会自动忽视通常不会使用到的那些文件（比如 CVS 目录）

GemPackageTask 在内部使用如下标识符生成一个 Rake 目标：

```shell
package_directory/gemname-gemversion.gem
```

之后可以从放置 Rakefile 文件的相同目录中调用这项任务：

```shell
rake pkg/MomLog-0.5.0.gem
```

#### 17.4.9 维护 Gem

更改完代码之后，从 Rakefile 运行 Rake，它会生成更新的 MomLog gem，之后登录到你的 RubyForge 账户，将 gem 上传到你项目中的“文件”区。在等待 RubyGems 自动将这个 gem 发布到中央 gem 仓库的过程中，可以撰写发布通告并张贴到 RubyForge 项目中。

```shell
% gem query -rn MomLog
```
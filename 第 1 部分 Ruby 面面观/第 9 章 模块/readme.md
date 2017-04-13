模块提供了两个主要的好处：

*   提供了命名空间（namespace）来防止命名冲突
*   实现了 `mixin` 功能

### 9.1 命名空间

使用模块机制。模块定义了一个 `namespace`（命名空间），它是一个沙箱（sandbox），方法和常量可以在其中任意发挥。

```ruby
module Moral
  VERY_BAD = 0
  BAd = 1
  def Moral.sin(badness)
    # ...
  end
end
```

模块常量的命名和类常量一样，都以大写字母开头。

如果第三方的程序想要使用这些模块，它可以简单地加载这两个文件（使用 Ruby 的 require 语句）并引用它们的完整名称（qualified name）

```ruby
require "moral"
wrongdoing = Moral.sin(Moral::VERY_BAD)
```

用模块名和句点来调用模块方法，使用模块名和两个冒号来引用常量。

### 9.2 Mixin

模块并没有实例，因为模块并不是类。不过，可以在类的定义中 `include`（包含）一个模块。当包含发生时，模块的所有实例方法瞬间在类中也可以使用了。它们被混入（mix in）了。实际上，所混入的模块其实际行为就像是一个超类。

```ruby
module Debug
  def who_am_i?
    # ...
  end
end
class Phonograph
  include Debug
  # ...
end
```

`include` 与文件无关。Ruby 语句只是简单地产生一个指向指定模块的引用。如果模块位于另一个文件中，在使用 `include` 之前，你必须使用 `require` 将文件加载进来。 Ruby 的 `include` 并非简单地将模块的实例方法拷贝到类中。相反，它建立一个由类到所包含模块的引用。如果多个类包含这个模块，它们都指向相同的内容。即使当程序正在运行时，如果你改变模块中一个方法的定义，所有包含这个模块的类都会表现出新的行为。

以标准的 Ruby mixin —— Comparable 为例。你可以使用 Comparable mixin 向类中添加比较操作符以及 `between?` 方法。Comparable 假定任何使用它的类都定义了 `<=>` 操作符。你可以定义一个方法 `<=>`，再包含 `comparable`，就可以免费得到 6 个比较函数：

```ruby
class Song
  include Comparable
  def initialize(name, artist, duration)
    @name = name
    @artist = artist
    @duration = duration
  end
  def <=>(other)
    self.duration <=> other.duration
  end
end
```

### 9.3 迭代器与可枚举模块

Ruby 收集（collection）类支持大量针对收集的各种操作：遍历、排序等等。你的类也可以支持所有这些出色的特性，需要做的是编写一个称为 each 的迭代器，由它依次返回收集中的元素。包含入 Enumerable，然后你的类瞬间支持诸如 `map`、`include?` 和 `find_all` 等操作。如果你收集的对象中使用了 `<=>` 方法实现了有意义的排序语义，你还可以得到诸如 `min`、`max`、`sort` 等方法。

### 9.4 组合模块

`inject` 方法，也是通过包含 `Enumerable` 来提供支持的。

```ruby
class VowelFinder
  include Enumerable
  def initialize(string)
    @string = string
  end
  def each
    @string.scan(/[aeiou]/) do |vowel|
      yield vowel
    end
  end
end
vf = VowelFinder.new("the quick brown fox jumped")
vf.inject {|v,n| v + n }	# ->	"euiooue"

module Summable
  def sum
    inject {|v,n| v + n}
  end
end

class VowelFinder
  include Summable
end
vf = VowelFinder.new("the quick brown fox jumped")
vf.sum	# ->	"euiooue"
```

#### 9.4.1 Mixin 中的实例变量

一个 mixin 中的实例变量可能会和其宿主类或其他 mixin 中的实例变量相冲突。多数时候，mixin 模块并不带有它们自己的实例数据——它们只是使用访问方法从客户对象中取得数据。但如果你要创建的 mixin 不得不持有它们自己的状态。确保这个实例变量具有唯一的名字，可以对系统中其他的 mixin 区别开来（比如使用模块名作为变量名的一部分）。或者，模块可以使用模块一级的散列表，以当前对象的 ID 作为索引，来保存特定于实例的数据，而不必使用 Ruby 的实例变量。

```ruby
module Test
  State = {}
  def state=(value)
    State[object_id] = value
  end
  def state
    State[object_id]
  end
end

class Client
  include Test
end

c1, c2 = Client.new, Client.new
c1.state, c2.state = "cat", "dog"
c1.state	# ->	"cat"
c2.state	# ->	"dog"
```

#### 9.4.2 解析有歧义的方法名

方法查找是如何处理的？

Ruby 首先会从对象的直属类（immediate class）中查找，然后是类所包含的 mixin，之后是超类以及超类的 mixin。如果一个类有多个混入的模块，最后一个包含的模块将会被第一个搜索。

### 9.5 包含其他文件

Ruby 有两个语句完成，每当 `load` 方法执行时，都会将指定的 Ruby 源文件包含进来。

```ruby
load "filename.rb"
```

更常见的是使用 `require` 方法来加载指定的文件，且只加载一次。

```ruby
require "filename"
```

被加载文件中的局部变量不会蔓延到加载它们所在的范围中。

`require` 有额外的功能：它可以加载共享的二进制库。两者都可以接受相对或绝对路径。如果指定了一个相对路径，它们将会在当前加载路径中（`load path` —— `$`）的每个目录下搜索这个文件。

使用 `load` 或 `require` 所加载的文件，当然也可以包含其他文件，而这些文件又可包含别的文件。`require` 是一个可执行的语句——它可能在一个 `if` 内。搜索路径也可以在运行时更改。只需将你希望的目录加入到 `$:` 数组中

因为 `load` 会无条件地包含源文件，你可以使用它来重新加载一个在程序开始执行后可能更改的源文件。


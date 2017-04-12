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


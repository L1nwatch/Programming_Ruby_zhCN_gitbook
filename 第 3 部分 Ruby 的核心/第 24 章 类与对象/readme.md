Ruby 只有一个底层的类和对象结构。

一个 Ruby 对象有三个部分：一组标志、一些实例变量以及相关联的类。Ruby 类是 Class 的一个对象，包括一个对象所具备的所有内容、外加方法列表和一个超类的引用（超类本身是另一个类）。Ruby 中的所有方法调用必须指派一个接收者（默认为 self，即当前对象）。

### 24.1 类和对象是如何交互的

对象引用类，而类引用零个或多个超类。

#### 24.1.1 你的基本的、日常对象

略

#### 24.1.2 什么是 Meta

Class 对象的 klass 成员没有指向任何有意义的东西。

>   ##### Metaclass 和 Singleton（单例）类
>
>   你可以称之为 metaclass，但是和 Smalltalk 不同，它并非是一个关于类的类（class of a class）；它是一个类的 singleton 类。
>
>   *   Ruby 中的每个对象都有自己的属性（方法、常量等等），在其他语言中被保存在类中。就如同每个对象都有其自己的类
>   *   为了处理每个对象的属性，Ruby 为每个对象提供了一个好像类的东西，某些时候被称为 singleton 类。
>   *   在当前的实现中，singleton 类是具有特别标志的类，与对象和它们的类不同。可以把它们作为虚拟类。
>   *   类的 singleton 类的行为如何 Smalltalk 的 metaclass

虚拟类在 Ruby 内被不同地对待。最明显的不同时对外界来说它们实际上是不可见的：它们永远不会出现在像 `Module#ancesstors` 或 `ObjectSpace.each_object` 等方法返回的对象列表中，而你也无法使用 `new` 来创建其实例。

#### 24.1.3 特定于 Object 的类

Ruby 允许你创建一个和特定对象绑定的类。

```ruby
a = "hello"
class << a
  def to_s
    "The value is '#{self}'"
  end
  def two_times
    self + self
  end
end
a.to_s			# ->	"The value is 'hello'"
a.two_times		# ->	"hellohello"
```

这个示例使用 `class <<obj` 定义，基本的意思是“为对象 obj 构建一个新类”。也可以将它改写为：

```ruby
a = "hello"
def a.to_s
  "The value is '#{self}'"
end
def a.two_times
  self + self
end
a.to_s		# ->	"The value is 'hello'"
a.two_times	# ->	"hellohello"
```

两者的效果是相同的：向对象 a 中添加一个类。有关 Ruby 的实现：创建虚拟类并插入作为 a 的直接类。a 的原有类 String 作为虚拟类的超类。

Ruby 中的类是永不关闭的；你可以随时打开一个类并添加新的方法。这对虚拟类也适用。如果对象的 klass 引用已经指向了一个虚拟类，则并不会创建一个新的虚拟类。

`Object#extend` 方法将其参数中的方法添加到调用该方法的接收者中，因此，如果需要的话，它也创建一个虚拟类。`obj.extend(Mod)` 基本和下面的等价：

```ruby
class << obj
  include Mod
end
```

#### 24.1.4 Mixin 模块

当类包含一个模块时，模块的实例方法就变为类的实例方法。好像模块变成类的超类。

当你包含一个模块时，Ruby 创建了一个指向该模块的匿名代理类，并将这个代理插入到实施包含的类中作为其直接超类。代理类包含有指向模块实例变量和实例方法的引用。

同一个模块可能被包含到不同的类中，并出现在许多不同的继承链中。不过，由于代理类的存在，它指向唯一的底层模块：改变模块中某个方法的定义，它会改变所有包含该模块的类，无论过去或未来。

如果包含了多个模块，它们按次序插入到继承链中。

如果模块本身包含了其他模块，代理类的链会加入到包含该模块的所有类中，每个直接或间接包含的模块都对应有一个代理。

#### 24.1.5 扩展对象

你可以使用 `Object#extend` 将一个模块混合到对象中：

```ruby
module Humor
  def tickle
    "hee, hee!"
  end
end
a = "Grouchy"
a.extend Humor
a.tickle	# ->	"hee, hee!"
```

如果你在一个类定义中使用它，模块的方法会变为类方法。这是因为调用 `extend` 和 `self.extend` 是等价的，因此方法被添加到 self 中，而在类定义中则是添加到类本身。

### 24.2 类和模块的定义

在如 C++ 或 Java 的语言中，类定义是在编译器处理的：编译器创建符号表，计算出需要分配多少存储空间，构造分发表（dispatch table）等。

Ruby 则不同。在 Ruby 中，类和模块的定义是可执行的代码。虽然是在编译期进行解析，但当遇到定义时，类和模块是在运行时创建的。（这对方法定义也是成立的。）这可以让你可以比传统语言更动态地架构你的程序。你可以在类要被定义时，而非每次使用类对象时，做出决定。

```ruby
module Tracing
  # ...
end
class MediaPlayer
  include Tracing if $DEBUG
  if ::EXPORT_VERSION
    def decrypt(stream)
      raise "Decryption not available"
    end
  else
    def decrypt(stream)
      # ...
    end
  end
end
```

类定义在执行时就是以这个类作为当前对象。

#### 24.2.1 类实例变量

对一般的实例变量来说，属性访问方法是在类一级定义的。对类实例变量来说，必须在 metaclass 中定义访问函数。

```ruby
class Test
  @cls_var = 123
  class << self
    attr_reader :cls_var
  end
end
Test.cls_var	# ->	123
```

许多我们在定义类或模块时使用的指令，例如 `alias_method`、`attr` 和 `public`，都是 Module 类中的方法——你可以通过编写 Ruby 代码来扩展类或模块定义的功能。

`once` 使用了这个特性，表明某个特定的方法体只应该被调用一次。

```ruby
class ExampleDate
  # ...
  once :as_string, :as_YMD
end
```

#### 24.2.2 类名是常量

所有的内建类，以及你所定义的类，都有一个对应的全局常量，它的名字和你的类名相同。

在系统中有两个名为 String 的东西。一个是指向 String 类的常量（Class 类的一个对象），另一个是（类）对象本身。

类名为常量的事实，意味着你可以把类和其他 Ruby 对象一样来对待：你可以拷贝它们，将它们作为参数传入方法，或者在表达式中使用它们。

```ruby
def factory(klass, *args)
  klass.new(*args)
end
factory(String, "Hello")	# ->	"Hello"
factory(Dir, ".")
```

如果一个无名的类被赋值给一个常量，Ruby 将常量作为类名。

### 24.3 顶层的执行环境

在顶层，我们在某个预定义的对象上下文中执行代码。当我们定义方法时，我们实际创建了 Object 类的一个（私有的）实例方法。

```ruby
puts "Hello, World"
self.class	# ->	Object
```

顶层的实例变量也属于这个顶层对象。

### 24.4 继承与可见性

在一个类定义中，你可以修改祖先类中方法的可见性。

```ruby
class Base
  def aMethod
    # xxx
  end
  private :aMethod
end
class Derived1 < Base
  public :aMethod
end
class Derived2 < Base
end
```

我们可以调用 Derived1 类实例中的 aMethod，但是不能通过 Base 或 Derived2 的实例来调用它。

如果一个子类改变了父类中某个方法的可见性，Ruby 实际上在子类中插入了一个隐藏的代理方法，使用 super 调用原有的方法。然后，它将这个代理的可见性设置为你所需要的。

```ruby
class Derived1 < Base
  def aMethod(*args)
    super
  end
  public :aMethod
```

### 24.5 冻结对象

任何对象都可以通过调用 `Object#freeze` 被冻结。一个被冻结的对象是不能修改的：你不能改变它的实例变量（无论直接或间接），你不能为其关联单例方法，而且，如果它是类或模块，你不能添加、删除或更改它的方法。一旦被冻结，对象将始终保持冻结状态：没有 `Object#thaw` 方法可用。你可以使用 `Object#frozen?` 来判断对象是否被冻结。

当你拷贝一个被冻结的对象时，如果你调用对象的 clone 方法，整个对象的状态（包括它是否被冻结）都会被拷贝到新对象中。另一方面，dup 通常只拷贝对象的内容——新拷贝不会继承冻结的状态。


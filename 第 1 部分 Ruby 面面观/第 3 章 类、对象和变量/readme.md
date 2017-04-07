在 Ruby 程序中，initialize 是一个特殊的方法。当你调用 `new` 创建一个新的对象时，Ruby 首先分配一些内存来保存未初始化的对象，然后调用对象的 `initialize` 方法，并把调用 `new` 时所使用的参数传入该方法。

`inspect` 方法（可以发送给任何对象）默认将对象的 ID 和实例变量格式化。

`inspect` 默认的格式化有时并不合用，Ruby 有一个标准消息（message）`to_s`，它可以发送给任何一个想要输出字符串表示的对象。

### 3.1 继承和消息

继承：

```ruby
class KaraokeSong < Song
end
```

如果要重写 `to_s`，让子类直接访问其祖先的实例变量，这是实现 `to_s` 的一种糟糕方式。直接戳进父类的内部结构，并且显式地检验它的实例变量，会使得我们和父类的实现紧密地绑在一起。

一个好的方式是使用 `super`，当你调用 `super` 而不使用参数时，Ruby 向当前对象的父类发送一个消息，要求它调用子类中的同名方法。

```ruby
class KaraokeSong < Song
  def to_s
    super + " [#@lyrics]"
  end
end
```

### 3.2 对象和属性

一个对象的外部可见部分被称为其**属性**。

#### 继承与 Mixin

某些面向对象语言（C++）支持多继承，而例如 Java 和 C# 仅支持单继承。Ruby 提供了一种折中，Ruby 类只有一个直接的父类，因此 Ruby 是一门单继承语言。不过，Ruby 类可以从任何数量的 mixin（类似于一种部分的类定义）中引入（include）功能。

`attr_reader`，以下两份代码的效果是等价的：

```ruby
class Song
  def name
    @name
  end
  
  def artist
    @artist
  end
end
# 等价
class Song
  attr_reader :name, :artist
end
```

`attr_reader` 实际上是 Ruby 的一个方法，而 `:name` 等符号（Symbol）是其参数，它会通过代码求解（code evaluation）动态地在 Song 类中加入实例方法体。

构成体 `:artist` 是一个表达式，它返回对应 `artist` 的一个 Symbol 对象。可以将 `:artist` 看作是变量 artist 的名字，而普通的 artist 是变量的值。

#### 3.2.1 可写的属性

在 Ruby 中，可以通过创建一个名字以等号结尾的方法来达成这一目标。这些方法可以作为赋值操作的目标。

```ruby
class Song
  def duration=(new_duration)
    @duration = new_duration
  end
end
```

同样，Ruby 又提供了一种快捷方式来创建这类简单的属性设置方法：

```ruby
class Song
  attr_writer :duration
end
```

#### 3.2.2 虚拟属性

这里使用属性方法创建一个虚拟的实例变量：

```ruby
class Song
  def duration_in_minutes
    @duration / 60.0
  end
  def duration_in_minutes=(new_duration)
    @duration = (new_duration * 60).to_i
  end
end
```

这被称为统一访问原则（Uniform Access Principle）。通过隐藏实例变量与计算的值的差异，可以向外部世界屏蔽类的实现。

#### 3.2.3 属性、实例变量及方法

### 3.3 类变量和类方法

#### 3.3.1 类变量

类变量由两个 `@@` 开头

类变量对类及其实例都是私有的，如果想让它们能够被外部世界访问，需要编写访问方法。

#### 3.3.2 类方法

类方法和实例方法是通过它们的定义区别开来的；通过在方法名之前放置类名以及一个句点，来定义类方法：

```ruby
class Example
  def instance_method		# instance method
  end
  def Example.class_method	# class method
  end
end
```

#### 3.3.3 单件与其他构造函数

实现 Singleton 模式，在《Design Patterns（设计模式）》中记载。

以下代码使得只有一种方式来创建日志对象，那就是调用 `MyLogger.create`，并且还保证只有一个日志对象被创建（但不保证线程安全）：

```ruby
class MyLogger
  private_class_method :new
  @@logger = nil
  def MyLogger.create
    @logger = new unless @@logger
    @@logger
  end
end
```

>   ### 类方法定义
>
>   实际上，你可以用很多方式来定义类方法
>
>   ```ruby
>   class Demo
>     def Demo.meth1
>       # ...
>     end
>     def self.meth2
>       # ...
>     end
>     class << self
>       def meth3
>         # ...
>       end
>     end
>   end
>   ```

使用类方法作为伪构造函数（pseudo-constructors），可以让使用类的用户更轻松些。

### 3.4 访问控制

在 Ruby 中改变一个对象的状态，唯一的简单方式就是调用它的方法。控制对方法的访问，就得以控制对对象的访问。Ruby 为你提供了三种级别的保护：

*   Public（公有）方法可以被任何人调用，没有限制访问控制。方法默认是 public 的（除了 initialize)
*   Protected（保护）方法只能被定义了该方法的类或其子类的对象所调用。整个家族均可访问
*   Private（私有）方法不能被明确的接收者调用，其接收者只能是 self

Ruby 和其他面向对象语言的差异，还体现在另一个重要的方面。访问控制是在程序运行时动态判定的，而非静态判定。只有当代码视图执行受限的方法，你才会得到一个访问违规。

#### 3.4.1 指定访问控制

如果使用时没有参数，这 3 个函数设置后续定义方法的默认访问控制。

```ruby
class MyClass
  def method1 # default is "public"
  end
  protected
  def method2	# protected
  end
  private
  def method3	# private
  end
  public
  def method4	# public
  end
end
```

 另外，还可以通过将方法名作为参数列表传入访问控制函数来设置它们的访问级别：

```ruby
class MyClass
  def method1
  end
  public	:method1, :method4
  protected	:method2
  private	:method3
end
```

### 3.5 变量

变量只是对象的引用。对象漂浮在某处一个很大的池中（大多数时候是堆，即 heap 中），并由变量指向它们。

赋值别名（alias）对象，潜在地给了你引用同一对象的多个变量。可以通过使用 `String` 的 `dup` 方法来避免创建别名，它会创建一个新的、具有相同的内容的 String 对象。

```ruby
person1 = "Tim"
person2 = person1.dup
```

可以通过冻结一个对象来阻止其他人对其进行改动。试图更改一个被冻结的对象，Ruby 将引发（raise）一个 TypeError 异常。

```ruby
person1 = "Tim"
person2 = person1
person1.freeze
person2[0] = "J"	# raise TypeError
```


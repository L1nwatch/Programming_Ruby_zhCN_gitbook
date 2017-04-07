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




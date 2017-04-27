缺乏静态类型对编写可靠的应用来说并不是一个问题。

在大多数主流语言中，静态类型系统在程序安全方面没有真正地起到太大作用。如果 Java 的类型系统可靠，则无须实现 `ClassCastException` 异常。但这个异常是需要的，因为 Java 中存在运行时的类型非确定性（与 C++、C# 等一致）。静态类型（typing）有助于代码优化，并通过工具提示（tooltip）帮助 IDE 做得更聪明些，但是没有证据表明它促进了编写更为安全可靠的代码。

### 23.1 类不是类型

对象的类型（type）是它的类（class）——所有对象都是某个类的实例，类（class）是对象的类型（type）。

即使在 Java 中，类并不总是类型——有时候类型是类的子集，并且有时候对象实现了多个类型。

`duck typing` 适合用来测试。

### 23.2 像鸭子那样编码

对象的类型是根据它能够做什么而不是根据它的类决定的。（因此 Ruby 1.8 目前不赞成使用 `Object#type` 方法而使用 `Object#class` 方法：这个方法返回接收者的类，因为 `type` 这个名字会让人产生误解）。

有时候你想在这种自由的（laissez-faire）编程风格之余，对参数进行检查。你会考虑是根据对象的能力而不是它的类来做这个检查。

```ruby
def append_song(result, song)
  unless result.respond_to?(:<<)
    fail TypeError.new("'result' needs '<<' capability")
  end
  unless song.respond_to?(:artist) && song.respond_to?(:title)
    fail TypeError.new("'song' needs 'artist' and 'title'")
  end
  
  result << song.title << " (" << song.artist << ")"
end
```

当然，在沿这条路走下去之前，确保你真正从中受益——需要编写和维护很多额外的代码。

### 23.3 标准协议和强制转换

解释器和标准库会使用各种各样的协议来处理其他语言在使用类型时遇到的一些问题。

Ruby 提供了转换协议（conversion protocols）的概念，对象可以选择把自己转换成另一个类的对象。Ruby 有三种标准的方式来实现它。

诸如 `to_s` 和 `to_i` 方法分别把它们的接收者转换成字符串和整数。这些转换方法不是非常严格的。

第二种形式的转换韩树使用名字如 `to_str` 和 `to_int` 的方法。它们是严格的转换函数。

少数几个严格的转换函数被內建到标准库中：`to_ary`、`to_hash`、`to_int`、`to_io`、`to_proc`、`to_str`、`to_sym`

#### 23.3.1 数字强制转换

在 Ruby 中有基于 coerce 方法的强制（coercion）协议。coerce 的基本操作是，它接受两个数字（一个是接受者，另一个是参数）。它返回包含两个元素的数组，分别是两个数字的表示（但是参数在前，后面跟着接受者）。`coerce` 方法保证两个对象会有相同的类，因此可以对它们进行算数运算。

```ruby
1.coerce(2)		# => [2, 1]
4.5.coerce(2)	# => [2.0, 4.5]
```

技巧在于接受者调用其参数的 `coerce` 方法来生成数组。这种技术被称为两次分发（double dispatch），它允许方法根据它的类并且还有参数的类来改变自身的行为。

假设正在编写信的类来进行算术运算。为了使用强制转换，需要实现 coerce 方法。它接收一些其他类型的数字作为参数，并返回一个数组，其中包含同一类的两个对象，它们的值与其参数及其自身相等。

```ruby
class Roman
  def initialize(value)
    @value = value
  end
  def coerce(other)
    if Integer === other
      [other, @value]
    else
      [Float(other), Float(@value)]
    end
  end
  # .. other Roman stuff
end
iv = Roman.new(4)
xi = Roman.new(11)
3 * iv 		# -> 12
1.1 * xi	# -> 12.1
```

最后，应当小心使用 `coerce` ——尝试总是强制转换到更通用的类型，否则可能最终造成强制转换循环。

### 23.4 该做的做，该说的说

略
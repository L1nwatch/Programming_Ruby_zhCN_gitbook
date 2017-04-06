### 2.1 Ruby 是一门面向对象语言

在 Ruby 中，通过调用构造函数（constructor）来创建对象，标准的构造函数被称为 `new`

```ruby
song = Song.new("Ruby Tuesday")
```

每个对象有一个唯一的对象标识符（object identifier，缩写为 object ID）。

方法是通过向对象发送消息（message）来唤起调用的。消息包含方法名称以及方法可能需要的参数。`.` 号之前的东西被称为接收者（receiver）

### 2.2 Ruby 的一些基本知识

下面两行是等同的，主要是指括号问题：

```ruby
puts say_goodnight("John-Boy")
puts(say_goodnight("John-Boy"))
```

但是考虑优先级的问题，最好是都使用括号

Ruby 对单引号串处理得很少。Ruby 对双引号字符串有更多的处理，首先它寻找以反斜线开始的序列，并用二进制值替换它们。

Ruby 对双引号字符串所做的第二件事情是字符串内的表达式内插（expression interpolation），`#{表达式}` 序列会被”表达式“的值替换。

为了方便起见，如果表达式只是一个全局实例或类变量，则不需要提供花括号：

```ruby
$greeting = "Hello"		# $greeting 是全局变量
@name	  = "Prudence"	# @name 是实例变量
puts "#$greeting,#$name"
```

Ruby 方法所返回的值，是最后一个被求值的表达式的值，所以可以把这个临时变量和 return 语句都去掉：

```ruby
def say_goodnight(name)
  "Good night, #{name}"
end
puts say_goodnight("Ma")
```

Ruby 使用一种命名惯例来区分名称的用途：名称的第一个字符显示这个名称如何被使用。局部变量、方法参数和方法名称都必须以小写字母或下画线开始。全局变量都有 `$` 为前缀，而实例变量以 `@` 符号开始，类变量以 `@@` 开始。类名称、模块名称和常量都必须以一个大写字母开始。

名称可以是字母、数字和下画线的任意组合（但跟在@符号之后的符号不能是数字）。按惯例，包含多个单词的实例变量名称在词与词之间使用下画线连接，包含多个单词的类变量名称使用混合大小写（每个单词首字母大写）。方法名称可以 `?`、`!` 和 `=` 字符结束

| 局部变量             | 全局变量        | 实例变量       | 类变量        | 常量和类名称      |
| ---------------- | ----------- | ---------- | ---------- | ----------- |
| name             | `$debug`    | `@name`    | `@@total`  | PI          |
| `fish_and_chips` | `$CUSTOMER` | `@point_1` | `@@symtab` | FeetPerMile |
| `x_axis`         | `$_`        | `@X`       | `@@N`      | String      |
| `thx1138`        | `$plan9`    | `@_`       | `@@x_pos`  | MyClass     |
| `_26`            | `$Global`   | `@plan9`   | `@@SINGLE` | JazzSong    |

### 2.3 数组和散列表

访问数组元素是高效的，但是散列表提供了灵活性。

在 Ruby 中，nil 是一个对象，与别的对象一样，只不过它是用来表示没有任何东西的对象。

在 Ruby 中，`%w` 可以让我们快速创建一组单词的数组

默认情况下，如果用一个散列表没有包含的键进行索引，散列表就返回 `nil`。当创建一个新的空散列表时，可以指定一个默认值：`histogram = Hash.new(0)`

### 2.4 控制结构

大多数 Ruby 语句会返回值，这意味着可以把它们当条件使用。例如，`gets` 方法从标准输入流返回下一行，或者当到达文件结束时返回 `nil`。

```ruby
while line = gets
  puts line.downcase
end
```

还有语句修饰符（statement modifiers）：

```ruby
puts "Danger, While Robinson" if radiation > 3000
```

### 2.5 正则表达式

可以使用如下的正则表达式来编写模式，它会匹配包含 Perl 或 Python 的字符串：

```ruby
/Perl|Python/
```

也可以使用括号：

```ruby
/P(erl|ython)/
```

sub 替换第一个匹配项，gsub 替换所有匹配项

### 2.6 Block 和迭代器

Block，一种可以和方法调用相关联的代码块。

可以用 block 实现回调，它只是在花括号或者 `do...end` 之间的一组代码

```ruby
{ puts "Hello" }	# this is a block
do	# and so is this
  club.enroll(person)
  person.socialize
end
```

Ruby 标准的一个约定俗成，单行 block 用花括号，多行用 `do/end`

一旦创建了 block，就可以与方法的调用相关联。把 block 的开始放在含有方法调用的源码行的结尾处，就可以实现关联。

```ruby
greet { puts "Hi" }
```

如果方法有参数，它们出现在 block 之前

使用 Ruby 的 yield 语句，方法可以一次或多次地调用（invoke）相关联的 block。

```ruby
def call_block
  puts "Start of method"
  yield
  yield
  puts "End of method"
end

call_back { puts "In the block" }
```

输出结果：

```ruby
Start of method
In the block
In the block
End of method
```

可以提供参数给对 `yield` 的调用：参数会传递到 block 中。在 block 中，竖线（|）之间给出参数名来接受这些来自 yield 的参数：

```ruby
def call_block
  yield("hello", 99)
end
call_block {|str, num| ... }
```

在 Ruby 库中大量使用了 block 来实现迭代器：

```ruby
animals = %w( ant bee cat dog elk )
animals.each {|animal| puts animal }
```

### 2.7 读/写文件

puts 输出它的参数，并在每个参数后面添加回车换行符。

print 也输出它的参数，但没有添加回车换行符。

printf，它在一个格式化字符串的控制下打印出它的参数。

gets 从程序的标准输入流中读取下一行。

Ruby 风格会使用迭代器和预定义对象 ARGF，ARGF 表示程序的输入文件：

```ruby
print ARGF.grep(/Ruby/)
```

可以用 `-w` 选项运行程序来显示警告信息

### 2.8 更高更远


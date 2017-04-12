Ruby 和其他语言的一个不同之处就是任何东西都能返回一个值：几乎所有东西都是表达式。

明显的一个好处是能够实现链式语句。

```ruby
a = b = c = 0				# ->	0
[3, 1, 7, 0].sort.reverse	# ->	[7, 3, 1, 0]
```

不太明显的好处是，C 和 Java 中的普通语句在 Ruby 中也是表达式。例如，if 和 case 语句都返回最后执行的表达式的值。

### 7.1 运算符表达式

实际上，Ruby 中的许多运算符都是由方法调用来实现的。例如，当你执行 `a*b+c` 时，等价于：

```ruby
(a.*(b)).+(c)
```

你可以重新定义任何不满足你需求的基本算术方法：

```ruby
class Fixnum
  alias old_plus +
    def +(ohter)
      old_plus(other).succ
    end
end
```

更有用的是，你写的类可以像內建对象那样参数到运算符表达式中：

```ruby
class Songe
  def [](from_time, to_time)
    # ...
  end
end
```

### 7.2 表达式之杂项

#### 7.2.1 命令展开

如果你用反引号（\`），或者以 `%x` 为前缀的分界形式，括起一个字符串，默认情况下它会被当做底层操作系统的命令来执行。表达式的返回值就是该命令的标准输出。你获得的返回值结尾可能会有回车符或者换行符。

你也可以在命令字符串中使用表达式展开和所有普通的转义序列。

```ruby
for i in 0..3
  status = 'dbmanager status id=#{1}'
  # ...
end
```

命令的退出状态（exit status）保存在全局变量 `$?` 中。

##### 重定义反引号

我们说反括号括起的字符串“默认”被当做命令来执行。实际上，字符串被传递给了名为 Kernel.\` 方法（单反引号）。如果你愿意，可以重载它。

```ruby
alias old_backquote `
def `(cmd)
  result = old_backquote(cmd)
  if $? != 0
    fail "Command #{cmd} failed: #$?"
  end
  result
end
```

### 7.3 赋值

赋值语句将左侧的变量或者属性（左值）设置为右侧的值（右值），然后返回该值作为赋值表达式的结果。这意味着你可以链接赋值语句，并可以在某些特殊的地方执行赋值操作。

```ruby
a = b = 1 + 2 + 3
# a ->	6
# b	->	6
a = (b = 1 + 2) + 3
# a ->	6
# b ->	3
File.open(name = gets.chomp)
```

Ruby 的赋值语句有两种形式。第一种是将一个对象引用赋值给变量或者常量。这种形式的赋值在 Ruby 语言中是直接执行的（hardwired）。

第二种形式等号左边是对象属性或者元素的引用。

```ruby
song.duration = 234
instrument["ano"] = "ccolo"
```

这种形式的特殊之处在于，它是通过调用左值的方法来实现的，这意味着你可以重载它们。只要简单地定义一个以等于号结尾的方法即可。这个方法以其右值作为它的参数。

这种设置属性的方法不必和内部的实例变量相对应，并且具有赋值方法的属性也并非必须要有读取该属性的方法。

```ruby
class Amplifier
  def volume=(new_volume)
    self.left_channel = self.right_channel = new_volume
  end
end
```

在老版本的 Ruby 中（低于 Ruby1.8），赋值语句的返回值是设置该属性的方法的返回值。在 Ruby 1.8 中，赋值语句的值总是参数的值而方法的返回值将被丢掉。

```ruby
class Test
  def val=(val)
    @val = val
    return 99
  end
end
t = Test.new
a = t.val = 2
a # -> 2
```

#### 7.3.1 并行赋值

Ruby 的赋值实际是以并行方式执行的，所以赋值语句右边的值不受赋值语句本身的影响。在左边的任意一个变量或属性被赋值之前，右边的值按它们出现的顺序被计算出来。

当赋值语句有多于一个左值时，赋值表达式将返回由右值组成的数组。如果赋值语句的左值多于右值，那么多余的左值将被忽略。如果右值多于左值，那么额外的右值将被忽略。如果赋值语句仅有一个左值，而有多个右值，那么右值将被转换成数组，然后赋值给左值。

>   ##### 在类中使用访问方法
>
>   可写属性有个隐藏的陷阱。通常，类中的方法可以通过函数形式（即带一个隐式 self 作为接受者）调用同一个类的其他方法和它的父类的方法。然而这不适用于属性赋值函数：Ruby 看到赋值语句时，会认为左边的名字是局部变量，而不是为一个属性赋值的方法调用。
>
>   ```ruby
>   class BrokenAmplifier
>     attr_accessor :left_channel, :right_channel
>     def volume=(vol)
>       # 注意 left_channel 仅仅是局部变量，这个赋值不会影响到 self.left_channel
>       left_channel = self.right_channel = vol
>     end
>   end
>   ```

使用 Ruby 的并行赋值操作，你可以叠起和展开数组：

```ruby
a = [1, 2, 3, 4]
b, c = a		# ->	b = 1, c = 2
b, *c = a		# ->	b = 1, c = [2, 3, 4]
b, c = 99, a	# ->	b = 99, c = [1, 2, 3, 4]
b, *c = 99, a	# ->	b = 99, c = [[1, 2, 3, 4]]
b, c = 99, *a	# ->	b = 99, c = 1
b, *c = 99, *a	# ->	b = 99, c = [1, 2, 3, 4]
```

##### 嵌套赋值

并行赋值还有一个值得一提的特性：赋值语句的左边可以含有一个由括号括起来的变量列表。Ruby 视这些变量为嵌套赋值语句。在处理更高层级的赋值语句前，Ruby 会提取出对应的右值，并赋值给括起来的变量。

```ruby
b, (c, d), e = 1, 2, 3, 4			# ->	b = 1, c = 2, d = nil, e = 3
b, (c, d), e = [1, 2, 3, 4]			# ->	b = 1, c = 2, d = nil, e = 3
b, (c, d), e = 1, [2, 3], 4			# ->	b = 1, c = 2, d = 3, e = 4
b, (c, d), e = 1, [2, 3, 4], 5		# ->	b = 1, c = 2, d = 3, e = 5
b, (c, *d), e = 1, [2, 3, 4], 5		# ->	b = 1, c = 2, d = [3, 4], e = 5
```

#### 7.3.2 赋值语句的其他形式

Ruby 有一个句法的快捷方式：`a = a + 2` 可以写成 `a += 2`。内部处理时，会将第二种形式先转换为第一种形式。

Ruby 不支持 C 或者 Java 中的自加（++）和自减（--）运算符。

### 7.4 条件执行

#### 7.4.1 布尔表达式

Ruby 对“真值”（truth）的定义很简单：任何不是 `nil` 或者常量 `false` 的值都为真。

然而，数字 0 不被解释为假值，长度为 0 的字符串也不是假值。

##### `Defined?`、与、或、非

Ruby 支持所有的标准布尔操作符，并引入了一个新操作符：`defined?`

Ruby 执行短路求解，`and` 和 `&&` 都可以表示与运算，唯一区别在于优先级不同（`and` 低于 `&&`，`or` 和 `||` 也是、`not` 与 `!` 也是）

需要注意 `and` 和 `or` 有相同的优先级，而 `&&` 的优先级高于 `||`

如果参数（可以是任意表达式）未被定义，`defined?` 操作符返回 `nil`；否则返回对参数的一个描述。如果参数是 `yield`，而且有一个 `block` 和当前上下文相关联，那么 `defined?` 返回字符串 `yield`

```ruby
defined? 1			# ->	"expression"
defined? dummy		# ->	nil
defined? printf		# -> 	"method"
defined? String		# ->	"constant"
defined? $_			# ->	"global-variable"
defined? Math::PI	# ->	"constant"
defined? a = 1		# ->	"assignment"
defined? 42.abs		# ->	"method"
```

Ruby 对象还支持如下比较方法：`==`、`===`、`<=>`、`=~`、`eql?` 和 `equal?`。除了 `<=>`，其他方法都是在类 `Object` 中定义的，但是经常被子类重载以提供适当的语义。

| 操作符               | 含义                                       |
| ----------------- | ---------------------------------------- |
| `==`              | 测试值相等与否                                  |
| `===`             | 用来比较 case 语句的目标和每个 when 从句的项             |
| `<=>`             | 通用比较操作符。根据接受者小于、等于或者大于其参数，返回 -1、0、+1     |
| `<`、`<=`、`>=`、`>` | 小于、小于等于、大于等于、大于比较操作符                     |
| `=~`              | 正则表达式模式匹配操作符                             |
| `eql?`            | 如果接受者和参数有相同的类型和相等的值，则返回真。`1==1.0` 返回真，但 `eql?(1.0)` 为假 |
| `equal?`          | 如果接受者和参数有相同的对象 ID，则返回真                   |

`==` 和 `=?` 都有相反的形式：`!=` 和 `!~`。不过，在 Ruby 读取程序的时候，会对它们进行转换：`a != b` 等价于 `!(a = b)`；`a !~ b` 等价于 `!(a =~ b)`。这意味着你写的类重载了 `==` 或者 `=~`，那么也会自动得到 `!=` 和 `!~`

还可以用 Ruby 的 range 作为布尔表达式。

Ruby 1.8 之前的版本中，可以用裸正则表达式（bare regular expression）作为布尔表达式。现在这种方式已经过时了，不过你仍然可以用 `~` 操作符将 `$_` 和一个模式进行匹配。

#### 7.4.2 逻辑表达式的值

操作符 `and`、`or`、`&&`、`||` 实际上返回首个决定条件真伪的参数的值。

Ruby 的一个惯用技法利用了这一特性：

```ruby
words[key] ||= []
words[key] << word
```

这两行代码将会把一个数组赋值给没有值的散列表元素，对已经有值的元素则原封不动。也可以写一行：

```ruby
(words[key] ||= []) << word
```

#### 7.4.3 If 和 Unless 表达式

Ruby 的 if 表达式：

```ruby
if song.artist == "Gillespie" then
  handle = "Dizzy"
  # ...
end
```

如果将 `if` 语句分布到多行上，那么可以不用 then 关键字：

```ruby
if song.artist == "Gillespie"
  handle = "Dizzy"
  # ...
end
```

还可以用冒号（：）来替代 `then`

`if` 是表达式而不是语句——它返回一个值：

```ruby
handle = if song.artist == "Gillespie" then
  "Dizzy"
  # ...
end
```

Ruby 还有一个否定形式的 if 语句

```ruby
unless song.duration > 180
  cost = 0.25
  # ...
end
```

Ruby 也支持 C 风格的条件表达式：

```ruby
cost = song.duration > 180 ? 0.35 : 0.25
```

#### If 和 Unless 修饰符

语句修饰符允许你将条件语句附加到常规语句的尾部。

因为 `if` 本身也是一个表达式，下面的用法会让人困惑：

```ruby
if artist == "John Coltrane"
  artist = "'Trane"
end unless use_nicknames == "no"
```

### 7.5 Case 表达式

Ruby 的 case 有两种形式。

第一种形式接近于一组连续的 `if` 语句：

```ruby
leap = case
when year % 400 == 0: true	# 冒号可以换成 then
when year % 100 == 0: false
else year % 4 == 0
end
```

第二种形式，在 `case` 语句的顶部指定一个目标，而每个 `when` 从句列出一个或者多个比较条件：

```ruby
case input_line
when "debug"
  dump_debug_info
  dump_symbols
when /p\s+(\w+)/
  dump_variable($1)
when "quit", "exit"
  exit
else
  print "Illegal command: #{input_line}"
end
```

case 返回执行的最后一个表达式的值。

`case` 通过比较目标和 when 关键字后面的表达式来运作。这个测试通过使用 `comparison === target` 来完成。只要一个类为 `===`（內建的类都有）提供了有意义的语义，那么该类的对象就可以在 case 表达式中使用。

正则表达式将 `===` 定义为一个简单的模式匹配

Ruby 的所有类都是类 Class 的实例，它定义了 `===` 以测试参数是否为该类或者其父类的一个实例。所以你可以测试对象到底属于哪个类：

```ruby
case shape
when Square, Rectangle
  # ...
when Circle
  # ...
when Triangle
  # ...
else
  # ...
end
```

### 7.6 循环

只要条件为真，`while` 循环就会执行循环体。`until` 循环与此相反，它执行循环体直到循环条件变为真。

你也可以用这两种循环做语句的修饰符：

```ruby
a = 1
a *= 2 while a < 100
a -= 10 until a < 100
```

range 可以作为某种后空翻：当某个事件发生时变为真，并在第二个事件发生之前一直保持为真。这常被用在循环中。比如读一个含有前十个序数的文本文件，但只输出匹配的行：

```ruby
file = File.open("ordinal")
while line = file.gets
  puts(line) if line =~ /third/ .. line =~ /fifth/
end
```

用在布尔表达式中的 range 的起点和终点本身也可以是表达式。每次求解总体布尔表达式时就会求解起点和终点表达式的值。变量 `$.` 包含当前的输入行号：

```ruby
File.foreach("ordinal") do |line|
  if (($. == 1) || line =~ /eig/) .. (($. == 3) || line =~ /nin/)
    print line
  end
end
```

当使用 while 和 until 做语句修饰符时，有一点需要注意：如果被修饰的语句是一个 `begin/end` 块，那么不管布尔表达式的值是什么，块内的代码至少会执行一次：

```ruby
print "Hello\n" while false
begin
  print "Goodbye\n"
end while false
```

#### 7.6.1 迭代器

几种循环方式：

```ruby
3.times { }
0.upto(9) {  }
0.step(12, 3) { }
[1, 1, 2, 3, 5].each { }
```

如果一个类支持 each 方法，那么也会自动支持 Enumerable 模块中的方法。例如，可以使用 Enumerable 中的 grep 方法，只迭代满足条件的行：

```ruby
File.open("ordinal").grep(/d$/) do |line|
  puts line
end
```

#### 7.6.2 For ... In

`for` 基本上是一个语法块，当我们写：

```ruby
for song in songlist
  song.play
end
```

Ruby 把它转换成：

```ruby
songlist.each do |song|
  song.play
end
```

`for` 循环和 `each` 形式的唯一区别是循环体中局部变量的作用域。可以使用 for 去迭代任意支持 each 方法的类。

```ruby
class Periods
  def each
    yield "Classical"
    yield "Jazz"
    yield "Rock"
  end
end

for genre in Periods.new
  print genre, " "
end
```

#### 7.6.3 Break, Redo 和 Next

break 终止最接近的封闭循环体，然后继续执行 block 后面的语句。redo 从循环头重新执行循环，但不重新计算循环条件表达式或者获得迭代中的下一个元素。next 跳到本次循环的末尾，并开始下一次迭代。

在 Ruby 1.8 中，可以传递一个值给 break 和 next。在传统的循环中，这可能只对 break 有意义，此时 break 将设置循环的返回值。（传递给 next 的任意值会被丢掉。）如果传统循环根本没有执行 break，那么它的值是 nil。

```ruby
result = while line = gets
  break(line) if line =~ /answer/
end
```

#### 7.6.4 Retry

redo 语句使得一个循环重新执行当前的迭代。但是有时你需要从头重新执行一个循环。retry 语句是它重新执行任意类型的迭代式循环。

retry 在重新执行之前会重新计算传递给迭代的所有参数：

```ruby
def do_until(cond)
  break if cond
  yield
  retry
end
i = 0
do_until(i > 10) do
  print i, " "
  i += 1
end
```

### 7.7 变量作用域、循环和 Blocks

while、until、for 循环內建到了 Ruby 语言中，但没有引入新的作用域：前面已存在的局部变量可以在循环中使用，而循环中新创建的局部变量也可以在循环后使用。

被迭代器使用的 block（loop 和 each）与此不同。通常，在这些 block 中创建的局部变量无法在 block 外访问

然而，如果执行 block 的时候，一个局部变量已经存在且与 block 中的变量同名，那么 block 将使用此已有的局部变量。因而，它的值在块后面仍然可以使用。

```ruby
x = nil
y = nil
[1, 2, 3].each do |x|
  y = x + 1
end
[x, y] # ->	[3, 4]
```

注意在外部作用域中变量不必有值：Ruby 解释器只需要看到它即可。

```ruby
if false
  a = 1
end
3.times { |i| a = i }
# a ->	2
```

目前，只能提几条建议以减少局部变量和 block 变量相互干扰带来的问题：

*   使你的方法和 block 尽量简短。变量越少，则它们相互干扰的机会就越少。而且也易于阅读，以检查是否有名字冲突
*   为局部变量和 block 参数使用不同的命名方式。


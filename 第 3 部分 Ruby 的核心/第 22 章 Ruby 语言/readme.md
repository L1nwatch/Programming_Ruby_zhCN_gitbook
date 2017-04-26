### 22.1 源代码编排

Ruby 程序是用 7 位 ASCII、Kanji（使用 EUC 或者 SJIS）或者 UTF-8 来表示的。如果要使用非 7 位 ASCII 的其他字符集，那么必须适当地设置 KCODE 选项。

Ruby 是基于行的语言，Ruby 的表达式和语句都以行尾结束，除非解析器能够确定语句是不完整的——比如行最后一个符号是操作符或者逗号。分号可以区分一行中的多个表达式。你也可以在行尾加一个反斜线表示延续到下一行。注释以 # 开始，到物理行结束为止。在语法分析时注释将被忽略。

Ruby 会忽略以 `=begin` 开头的和以 `=end` 开头的行之间的物理行，它们可以被用来注释掉一段代码或者用在源代码中嵌入文档。

Ruby 一次性读入整个程序，所以你可以用管道将程序传递给 Ruby 解释器的标准输入流。

```shell
echo 'puts "Hello"' | ruby
```

在源代码的任意位置，如果 Ruby 遇到只含有 `__END__` 且后没有空格的行，那么 Ruby 将认为该行是程序的结束行——后面的行都不会被当做程序代码。不过，使用全局的 IO 对象 DATA，可以将这些行读入到运行的程序中。

#### 22.1.1 BEGIN 和 END Blocks

Ruby 的每个源代码文件都可以声明当自己被装载时要执行的代码 block（BEGIN block）和程序运行结束后要执行的 block（END block）。

```ruby
BEGIN {
  # 开始代码
  }

END {
  # 结束代码
  }
```

一个程序可以包含多个 BEGIN 和 END block。BEGIN block 的执行顺序和它出现的顺序相同。END block 的执行顺序与此相反。

#### 22.1.2 常规分隔输入

除了常用的引用（quoting）机制外，还可以用其他形式来表示字符串字面量（literal）、数组、正则表达式和 shell 命令，这就是常规分隔语法。所有这些字面量都是以一个百分号开头，后跟一个指明字面量类型的字符的。

类型字符后面的是分隔字符，分隔字符可以是任意非字母或非多字节的字符。如果分隔符是`(`、`[`、`{` 或 `<` 其中之一，那么从它到对应的闭合分隔符之间的字符，属字面量所有，同时嵌套的分隔符对也考虑在内。对于其他的分隔符形式，在分隔符下一次出现之前的所有字符，属字面量所有。

```ruby
%q/this is a string/
%q-string-
%q(a (nested) string)
```

| 类型     | 意义       |
| ------ | -------- |
| %q     | 单引号字符串   |
| %Q，%   | 双引号字符串   |
| %w, %W | 字符串数组    |
| %r     | 正则表达式模式  |
| %x     | Shell 命令 |

分隔字符可以跨越多行：行结束符以及后续行开始处的空格，都将被包含到字符串中。

```ruby
meth = %q{def fred(a)
			a.each {|i| puts i}
		  end}
```

### 22.2 基本类型

Ruby 的基本类型包括数字、字符串、数组、散列表（hash）、区间（range）、符号和正则表达式。

#### 22.2.1 整数和浮点数

Ruby 的整数是 Fixnum 类或 Bignum 类的对象。Fixnum 对象可以容纳比本机字长少一位的整数。当一个 Fixnum 超过这个范围时，它将会自动转换成 Bignum 对象，Bignum 对象的表示范围仅受可用内存大小的限制。如果 Bignum 对象的操作结果可以用 Fixnum 表示，那么结果将以 Fixnum 类型返回。

整数由一个可选的符号标记、一个可选的进制指示符（0 代表八进制，0d 代表十进制，0x 代表十六进制，0b 代表二进制）和一个相应进制的字符串组成。

你可以在一个 ASCII 字符前加一个问号来获得其对应的整数值。Ctrl 组合键字符可以由 `?\C-x` 和 `?\cx` 来产生（Ctrl + x）。Meta 字符可以由 `?\M-x` 来生成。Meta 和 Ctrl 的组合键可以由 `?\M-\C-x` 生成。使用 `?\\` 序列你能得到反斜线字符的整数值。

一个带有小数点和/或指数的数字字面量被认为是 Float 对象，Float 对象和本机上的 double 数据类型大小一样。小数点后必须跟一个数字，因为像 `1.e3` 这样的串将试图调用 `Fixnum` 类的 `e3` 方法。Ruby 1.8 还要求在小数点前至少要有一个数字。

```ruby
12.34		# ->	12.34
-0.1234e2	# ->	-12.34
1234e-2		# ->	12.34
```

##### 字符串

单引号引起来的字符串字面量（例如 `'stuff'` 和 `%q/stuff/` ），会将 `\\` 序列转换为单个反斜线，还会将 `\'` 转换为单引号。所有其他的反斜线都不进行转换。

```ruby
%q(nesting (really) works)
%q no_blanks_here ;
```

双引号字符串（`"stuff"`、`%Q/stuff/` 和 `%/stuff/` ）还执行额外的替换。

字符串可以跨行，在这种情况下串包含换行符。也可以使用 `here documents` 来表示很长的字符串字面量。每当 Ruby 遇到 `<<identifier` 或者 `<<quoted string` 时，将用后续的逻辑输入行来生成字符串字面量，直到遇到以 identifier 或者 quoted string 开头的行时才结束。

你也可以在 `<<` 字符后紧跟一个减号，让终止符可以从左侧页空白缩进。如果终止符是由引号引起来的字符串，那么该引号对应的替换规则将被应用到 `here document`，否则使用双引号替换规则。

```ruby
print <<HERE
Double quoted \
here document.
It is #{Time.now}
HERE
# output
Double quoted here document.
It is 2017-04-24 15:03:06 +0800

print <<-'THERE'
This is a single quoted.
The above used #{Time.now}
THERE
# output
This is a single quoted.
The above used #{Time.now}
```

输入中相连的单引号字符串和双引号字符串将被连接成一个 String 对象。

字符串被保存在 8 位字节序列中，每个字节可以含有 256 个 8 位值中的任意一个，包括 null 和换行符。

当字符串字面量用于赋值语句或者作为参数时，一个新的 String 对象将被创建。

```ruby
3.times do
  print 'hello'.object_id, " "
end
```

#### 22.2.2 区间

除了用在条件表达式中，`expr..expr` 和 `expr...expr` 还能构成 Range 对象。两个点的形式是闭合区间，而三点的形式是半开半闭的。

#### 22.2.3 数组

数组类尾部的逗号将被忽略。

数组也可以用简写形式 %w 和 %W 来构成。%w 将空格隔开的 token 提取为连续的数组元素。在单个字符串内不执行替换。%W 对每个词执行和双引号字符串一样的替换规则。词之间的空格可以用反斜线转义。

```ruby
irb(main):001:0> arr = %w( fred wilma barney betty great\ gazoo )
=> ["fred", "wilma", "barney", "betty", "great gazoo"]
irb(main):002:0> arr = %w( Hey!\tIt is now -#{Time.now}-)
=> ["Hey!\\tIt", "is", "now", "-\#{Time.now}-"]
irb(main):003:0> arr = %W( Hey!\tIt is now -#{Time.now}-)
=> ["Hey!\tIt", "is", "now", "-2017-04-26 10:52:14 +0800-"]
```

#### 22.2.4 散列表

Ruby 的 Hash 中，键和值之间由 `=>` 序列分隔。

```ruby
colors = {
  "red" => 0xf00,
  "green" => 0x0f0,
  "blue" => 0x00f
}
```

##### 对散列表键的要求

散列表键必须能够响应 hash 消息并返回一个散列码（hash code），且某个键对应的散列码不能改变。散列表中使用的键也必须能用 `eql?` 来比较。

如果你保存了键对象的一个外部引用，并使用该引用修改了对象，那么这也将修改它的散列码，进而基于该键的查询将无效。

Ruby 会对字符串键进行特别处理。如果你使用 String 对象作为 hash 键，则 hash 将在内部复制该字符串，并使用该拷贝作为键。此拷贝将被冻结，对原字符串的任意改变都不会影响 hash。

如果你实现了自定义的类，并用该类的对象实例作为 hash 键，那么你必须确保：

*   一旦对象被创建，它的散列码就不再改变
*   每当键的散列码发生变化时都调用 `Hash#rehash` 方法重新对散列表进行索引

#### 22.2.5 符号

Ruby 符号是一个对应字符串的标识符。你可以通过在名字前加一个冒号来构建该名字的符号，也可以在任意字符串字面量前加一个冒号来创建该字符串的符号。在双引号字符串中会发生替换。

一个具体的名字或者字符串总是产生同样的符号。

```ruby
:Object
:my_variable
:"Ruby rules"
a = "cat"
:'catsup'	# ->	:catsup
:"#{a}sup"	# ->	:catsup
:'#{a}sup'	# ->	:"\#{a}sup"
```

其他语言称这个过程为 interning，而称符号为原子。

#### 22.2.6 正则表达式

正则表达式的字面量（literal）是 Regexp 类型的对象。可以通过调用 `Regexp.new` 构造函数显式创建正则表达式，也可以使用字面量形式 `/pattern/` 和 `%r{pattern}` 隐式创建。`%r` 构造体（contruct）是常规分隔输入的一种形式。

```ruby
/pattern/
/pattern/options
%r{pattern}
%r{pattern}options
Regexp.new('pattern' [, options])
```

##### 正则表达式选项

*   i：大小写无关。（通过设置 `$=` 来使匹配大小写无关的方式已经过时了）
*   o：替换一次。正则表达式中的任意 `#...` 替换仅在第一次求解（evalute）它的时候执行替换。否则，替换在每次字面量生成 Regexp 对象时执行
*   m：跨行模式。通常，"." 匹配换行符之外的任意字符。使用 `/m` 选项，"." 将匹配任意字符。
*   x：扩展模式，复杂的正则表达式可读性较差。x 选项允许你向模式中插入空格、换行符和注释以提高它的可读性。

还有另外一组选项可以设置正则表达式的语言编码。如果没有这些选项，那么将使用解释器的默认编码（可以用 `-K` 后者 `$KCODE` 设置）

*   n：没有编码（ASCII）
*   e：EUC
*   s：SJIS
*   u：UTF-8

##### 正则表达式模式

包括常规字符、`^`、`&`、`\A`、`\z`、`\Z`、`\b`、`\B`、`\G`、、`[characters]`、`\d, \s, \w`、`\D, \S, \W`、`.`、`re*`、`re+`、`re{m, n}`、`re{m,}`、`re{m}`、`re?`、`re1|re2`、`(...)`

锚点 `\G` 用在全局匹配方法 `String#gsub`、`gsub!`、`index` 和 `scan` 中。在重复匹配中，它代表字符串内迭代最后一个匹配结束的位置。开始时 `\G` 指向字符串开始的位置（或 `index` 第二个参数引用的字符）

```ruby
irb(main):004:0> "a01b23c45 d56".scan(/[a-z]\d+/)
=> ["a01", "b23", "c45", "d56"]
irb(main):005:0> "a01b23c45 d56".scan(/\G[a-z]\d+/)
=> ["a01", "b23", "c45"]
```

##### 替换

*   `#{...}`，执行表达式替换。默认情况，每次求解正则表达式时都执行该替换。如果设置了 `/o` 选项，那么仅在第一次求解时执行替换。
*   `\0`、`\1`、`\2` ... `\9`、`\&`、\\\`、`\'`、`\+`，替换第 n 组子表达式匹配的值，或者整个匹配、匹配之前、匹配之后，或者最高的那一组

##### 正则表达式扩展

Ruby 的正则表达式也提供了传统 Unix 正则表达式之外的一些扩展。所有这些扩展都位于字符 `(?` 和 `)` 之间。括起这些扩展的括号是组，但它们不生成后向引用（back reference）：它们不设置 `\1` 和 `$1` 等的值。

*   `(?# comment)`：插入注释到模式中。当模式匹配时，其中的内容将被忽略
*   `(?:re)`：使 re 成为组，但不产生后向引用。当你想对结构进行分组，但又不想为该组产生 `$1` 或者任何其他特殊变量时很有用。

```ruby
irb(main):005:0> date = "12/25/01"
=> "12/25/01"
irb(main):006:0> date =~ %r{(\d+)(/|:)(\d+)(/|:)(\d+)}
=> 0
irb(main):008:0> [$1,$2,$3,$4,$5]
=> ["12", "/", "25", "/", "01"]
irb(main):009:0> date =~ %r{(\d+)(?:/|:)(\d+)(?:/|:)(\d+)}
=> 0
irb(main):010:0> [$1,$2,$3]
=> ["12", "25", "01"]
```

*   `(?=re)`：在此处匹配 re，但并不耗用（consume）它（也被称为零宽度正向查看 zero-width positive lookahead）。这可以让你向前寻找一个匹配的上下文而不影响 `$&`。

下面的例子，scan 方法匹配后跟逗号的词，但逗号不被包括在匹配的结果中。

```ruby
str = "red, white, and blue"
str.scan(/[a-z]+(?=,)/)	# -> ["red", "white"]
```

*   `(?!re)`：如果此处不匹配 re，则认为匹配。且不消耗匹配（零宽度正向查看）。例如 `/hot(?!dog)(\w+)/` 与含有字符 hot 且后不跟 dog 的词相匹配，并保存词的结尾部分到 `$1` 中
*   `(?>re)`：在当前锚点的第一个正则表达式中嵌套一个独立的正则表达式。被其耗用的字符，就不会被上层的正则表达式访问到。因此这个结构可以抑制回溯，从而提高性能。例如，模式 `/a.*b.*a/` 在匹配含有一个 a 后跟多个 b 但并不以 a 结尾的字符串时，需要指数级时间复杂度。不过有时候这可以通过使用嵌套的正则表达式 `/a(?>.*b).*a/` 来避免。在这种形式中，嵌套的表达式将耗用掉所有的输入字符直到最后一个可能的 b。当检查尾部 a 匹配失败时，就没必要回退，并且模式匹配将迅速以失败告终（当匹配不应当直接查到最后一个 b 时，这个模式和原来的模式有不同的语义）。

```ruby
require "benchmark"
include Benchmark
str = "a" + ("b" * 5000)
bm(8) do |test|
  test.report("Normal:") { str =~ /^a.*b.*a/ }
  test.report("Nested:") { str =~ /^a(?>.*b).*a/ }
end
```

*   `(?imx)`：开启 i，m 或者 x 选项。如果用在组中，则只影响该组。
*   `(?-imx)`：关闭 i，m 或者 x 选项。
*   `(?imx:re)`：为 re 开启选项 i，m 或者 x。
*   `(?-imx:re)`：为 re 关闭选项 i，m 或者 x。

### 22.3 名字

一个名字是一个大写字母、小写字母或者下画线，后跟任意个命名用字符：大写字母、小写字母、下画线或者数字的任意组合。

全局变量名也可以由 `$-` 后跟一个字符或者下画线组成。后面的这些变量通常用来反映命令行选项的相应设置。

```shell
$params	$PROGRAM	$!	$_	$-a	$-K
```

#### 22.3.1 变量/方法二义性

当 Ruby 解析源代码文件时，它会记录所有已经被赋值的符号。它认为这些符号是变量。以后当遇到一个既可以是变量又可以是方法调用的符号时，Ruby 会检查是否已经对该符号进行了赋值。如果是，那么把该符号当做变量，否则当做方法调用。

```ruby
def a
  print "Function 'a' called\n"
  99
end
for i in 1..2
  if i == 2
    print "a=", a, "\n"
  else
    a = 1
    print "a=", a, "\n"
  end
end

## output
#a=1
#Function 'a' called
#a=99
```

注意赋值语句不一定被执行——只要 Ruby 看到了它就可以。

```ruby
a = 1 if false; a
```

### 22.4 变量和常量

Ruby 的变量和常量含有对象的引用。变量本身没有内在的类型。变量的类型仅仅由变量引用的对象所能响应的消息决定。

Ruby 常量也是对对象的引用。常量在它第一次被赋值的时候创建（通常是在类或模块的定义中）。Ruby 允许你改变常量的值，这会导致警告。

注意尽管应当不改变常量的值，但是可以改变它所引用的对象的内部状态。

```ruby
MY_CONST = "Tim"
MY_CONST[0] = "J"
MY_CONST	# ->	"Jim"
```

赋值会潜在地为对象定义别名，使得一个对象有不同的名字。

#### 22.4.1 常量和变量的作用域

在类或者模块内的任意位置都可以直接访问此类或模块中定义的常量。在类或者模块之外，可以通过在域作用符 `::` 前面加上一个返回适当类或者模块对象的表达式来访问。不属于任意类或者模块的常量，可以直接访问或者使用不带前缀的域作用符来访问。常量可以在方法体外定义。通过在常量名之前使用类或者模块名和域作用符，还可以将常量从外面添加到已存在的类或者模块中。

```ruby
OUTER_CONST = 99
class Const
  def get_const
    CONST
  end
  CONST = OUTER_CONST + 1
end
Const.new.get_const	# ->	100
Const::CONST		# -> 	100
::OUTER_CONST		# ->	99
Const::NEW_CONST = 123
```

在顶层使用的类变量在 Object 中定义，这种类变量类似于全局变量。在 singleton 方法中定义的类变量属于顶层类（这种使用方式已经过时了，会产生警告）。在 Ruby 1.9 中，类变量将是其类的私有变量。

局部变量的独特之处在于它们的作用域是静态确定的，然而却是动态创建的。

局部变量是在程序执行时为其第一次赋值的时候动态创建的。

引用一个在作用域内但未创建的局部变量会产生 NameError 异常。

Block 可以使用在它创建时已经存在的局部变量。这形成绑定的一部分。注意，尽管此时变量的绑定是固定的，但是在执行的时候，block 可以访问这些变量的当前值。即使原本外围的作用域被销毁了，绑定依然会保持这些变量。

`while`、`until` 和 `for` 的循环体和循环体外的代码属于同一作用域；前面已经存在的局部变量可以在循环中使用，而且新创建的任意局部变量也可以在循环体后继续使用。

#### 22.4.2 预定义变量

略

#### 22.4.3 全局常量

| 常量                | 类型         | 说明                                       |
| ----------------- | ---------- | ---------------------------------------- |
| DATA              | IO         | 如果主程序文件含有 `__END__` 指令，那么常量 DATA 将会被初始化为源代码中 `__END__` 之后的行 |
| FALSE             | FalseClass | 和 false 同义                               |
| NIL               | NilCLass   | 和 nil 同义                                 |
| RUBY_PLATFORM     | String     | 运行程序的平台标识符。这个字符串和 GNU configure 工具用的平台标识符有相同的形式 |
| RUBY_RELEASE_DATE | String     | 版本发布的日期                                  |
| RUBY_VERSION      | String     | 解释器的版本号                                  |
| STDERR            | IO         | 程序实际的标准错误流。初始值为 `$stderr`                |
| STDIN             | IO         | 程序实际的标准输入流。初始值为 `$stdin`                 |
| STDOUT            | IO         | 程序实际的标准输出流。初始值为 `$stdout`                |
| SCRIPT_LINES__    | Hash       | 如果常量 `SCRIPT_LINES__` 被定义，并引用一个 Hash，那么 Ruby 会保存它解析的每个文件内容为 Hash 的一项，键是文件的名字而值是一个字符串数组。 |
| TOPLEVEL_BINDING  | Binding    | 一个 Binding 对象表示在 Ruby 顶层上的绑定——程序被初始执行的层次。 |
| TRUE              | TrueClass  | 和 true 同义。                               |

 常量 `__FILE__` 和变量 `$0` 常常被合用，以使得仅当文件直接由用户执行时才运行文件中的代码。

```ruby
# Library code
# ...
if __FILE__ == $0
  # tests...
end
```

### 22.5 表达式

表达式中用的术语可以是下面任意一个：

*   字面量
*   Shell 命令
*   符号产生器（名字前加冒号）
*   变量引用或常量引用
*   方法调用

#### 22.5.1 操作符表达式

表达式可以用操作符来组合，不是每一个操作符都是由方法实现的，所以不是每一个操作符都可以被 override

#### 22.5.2 关于赋值的更多内容

如果左值是一个变量名或者常量名，那么该变量或者常量获得相应左值的引用。

如果左值是一个对象属性，那么接收对象中对应的属性设置函数将被调用，并以右值作为参数。

如果左值是数组元素引用，那么 Ruby 将调用接收对象的元素赋值操作符（`[]=`），并以方括号的下标和右值为参数。

##### 并行赋值

*   如果最后一个右值前面有一个星号，且实现了 `to_arry` 函数，那么该右值将被数组的元素取代，每个元素形成一个独立的右值。

#### 22.5.3 Block 表达式

略

#### 22.5.4 布尔表达式

##### 布尔表达式中的 Ranges

```ruby
if expr1 .. expr2
while expr1 ... expr3
```

在布尔表达式中使用的 `range` 具有双态触发器（flip-flop）的行为。它有两种状态：开和关，而且初始状态是关。每调用一次 range 变迁一次状态。如果当调用结束时状态机处于开状态，则 range 表达式返回 true；否则返回 false。

两点的 range 当第一次从关变成开时，它会立即求解结束条件，并相应地变迁状态。

三点形式的 range 不会在进入开状态后立即求解结束条件。

##### 布尔表达式中的正则表达式

在 Ruby 1.8 之前，布尔表达式中的单个正则表达式将会和变量 `$_` 的当前值进行匹配。现只有在命令行 `-e` 参数中出现的条件才支持这种行为。在通常的代码中，对隐含操作符和 `$_` 的使用已经慢慢废止了，最好使用匹配变量的显式匹配。如果需要匹配 `$_`，则使用：

```ruby
if ~/re/ ...	或	if $_ =~ /re/ ...
```

#### 22.5.5 if 和 Unless 表达式

略

#### 22.5.6 三元运算符

略

#### 22.5.7 case 表达式

Ruby 有两种形式的 case 语句。第一种会求解一系列条件的值，并执行第一个为真的条件对应的代码。

第二种形式的 case 表达式，在 case 关键字后面有一个目标表达式。

被比较目标可以是跟在星号后的一个数组引用，在这种情况下，在执行测试之前，数组会扩展成其元素。当比较返回真时，停止搜索，并执行与该比较目标相关联的代码体（不需要 break）。

`then` 关键字（或冒号）分隔 `when` 比较目标和它的代码体，如果代码体从一新行开始，则不需要 `then` 和冒号。

### 22.6 方法定义

使用 `constant.methodname` 形式或者更一般的 `(expr).methodname` 形式作为方法名，将创建一个与常量或者表达式引用的对象相关联的方法；该方法只能通过以表达式引用的对象作为接收者来调用。这种定义风格将创建单个对象的方法或称单例（singleton）方法。

```ruby
class MyClass
  def MyClass.method	# 定义
  end
end
MyClass.method			# 调用

obj = Object.new
def obj.method			# 定义
end
obj.method				# 调用

def (1.class).fred		# 接受者可以是表达式
end
Fixnum.fred				# 调用
```

方法定义可能不包含类或模块定义。它们可以含有嵌套的实例或者单例方法定义。当执行外部的方法时，内部的方法被定义。在被嵌套方法的上下文中，内部方法不是一个闭包（closure）——它是自包含的。

```ruby
def toggle
  def toggle
    "subsequent times"
  end
  "first time"
end

toggle	# ->	"first time"
toggle	# ->	"subsequent times"
toggle	# ->	"subsequent times"
```

#### 22.6.1 方法参数

表达式是从左向右求解的。表达式可以引用参数列表中它前面已定义的参数。

```ruby
def options(a=99, b=a+1)
  [a, b]
end
options		# ->	[99, 100]
options 1	# ->	[1, 2]
options 2,4	# ->	[2, 4]
```

可选的 block 参数必须是参数列表中的最后一个。无论何时调用该方法，Ruby 都会检查是否有关联的 block。如果有 block，它将被转换成 Proc 类的对象，然后赋值给 block 参数。如果没有 block，该参数将被设置为 `nil`。

### 22.7 调用方法

形参后面可以有一个 `key => value` 配对的列表。这些 `key => value` 对将被收集到一个新的 Hash 对象中，并作为一个形参传入方法。

Ruby 使用全局函数 `Kernel.block_given?` 的值来判断是否存在与本调用相关联的 block。

被包含的模块的实例方法看起来像是包含它们的类的匿名函数。如果没找到方法，Ruby 将调用接受者的 `method_missing` 方法。`Kernel.method_missing` 定义的默认行为是报告错误并终止程序。

当显式指明方法调用的接受者时，可以使用句点 `.` 或两个冒号 `::` 把它和方法名分隔开。这两种形式的唯一区别仅在方法名义大写字母开始时才会出现。在这种情况下，Ruby 会认为 `receiver::Thing` 方法调用是试图访问接受者中的称为 Thing 的常量，除非方法调用带有一个包含在括号中的参数列表。

```ruby
Foo.Bar()	# 方法调用
Foo.Bar		# 方法调用
Foo::Bar()	# 方法调用
Foo::Bar	# 访问常量
```

如果不带参数调用 `return`，则 `return` 的返回值为 `nil`。

#### 22.7.1 super

在方法体内，调用 super 就像调用原方法一样，不过是在含有原方法的对象的超类中搜索方法体。如果没有参数（且未加括号）传递给 super，则原方法的参数将作为 super 的参数，否则，super 的参数将被传递。

#### 22.7.2 操作符方法

如果操作符表达式中的操作符是一个可重新定义的方法，Ruby 像调用如下表达式那样执行操作符表达式。

```ruby
(expr1).operator() 或	(expr1).operator(expr2)
```

#### 22.7.3 属性赋值

```ruby
receiver.attrname = rvalue
```

这种赋值语句的返回值总是 rvalue 的值——`attrname=` 的返回值将被丢弃。如果你想访问返回值（多半情况下不是 rvalue 的值），需要向方法发送一个显示的消息。

```ruby
class Demo
  attr_reader :attr
  def attr=(val)
    @attr = val
    "return value"
  end
end
d = Demo.new
# 在如下情形中，@attr 都被设置为 99
d.attr = 99			# ->	99
d.attr=(99)			# ->	99
d.send(:attr=, 99)	# ->	"return value"
d.attr				# ->	99
```

#### 22.7.4 元素引用操作符

略

### 22.8 别名

```ruby
alias new_name old_name
```

将创建一个引用已有的方法、操作符、全局变量或正则表达式向后引用（`$&`、$\`、`$'`、`$+`）的新名字。局部变量、实例变量、类变量和常量不能有别名。alias 的参数可以是名字或符号。

当为方法起别名时，新的名字将指向原方法体的一个拷贝。即使后来方法被重新定义了，别名仍旧会调用原来的方法实现代码。

### 22.9 类定义

```ruby
class [scope::]classname[<superexpr]
  # body
end
class << obj
  # body
end
```

第一种形式中，一个命名类将被创建或者扩展。第二种形式中，一个匿名（单例）类会和指定的对象相关联。

如果 `superexpr` 存在，那么它应当是一个以 Class 对象为结果的表达式，而且它也将是被定义的类的超类。如果省略了 `superexpr`，则默认为类 `Object`。

在方法体内，随着各种定义代码的读入，大多数 Ruby 表达式将被执行。然而：

*   方法定义将在类的对象的一个表中注册该方法
*   嵌套的类和模块定义将被存储在类的常量中，而不是全局常量中。在定义嵌套的类和模块的类外可以通过使用 `::` 修饰其名字来访问它们。

```ruby
module NameSpace
  class Example
    CONST = 123
  end
end
obj = NameSpace::Example.new
a = NameSpace::Example::CONST
```

*   方法 `Module#include` 将把命名的模块添作被定义的类的匿名超类。

使用域作用符（：：）可以为类定义中的 classname 前置一个已存在的类或模块名。这种语法会将新的定义插入到前面已定义的模块和/或类的名字空间中，但不是在这些外部类的作用域中解释此定义。前面带有域作用符的 classname 将使其类或模块处于顶层作用域中。

在下面的例子中，类 C 被插入到模块 A 的作用域，但并不在 A 的上下文中进行解释。结果，对 CONST 的引用被解释成该名字对应的顶层常量而不是 A 的常量。而且我们必须使用单例方法的全名，因为在 `A::C` 的上下文中，C 本身不是一个已知的常量。

```ruby
CONST = "outer"
module A
  CONST = "inner"	# This is A::CONST
end
module A
  class B
    def B.get_const
      CONST
    end
  end
end
A::B.get_const	# ->	"inner"

class A::C
  def (A::C).get_const
    CONST
  end
end
A::C.get_const	# ->	"outer"
```

类定义是可执行的代码。在类定义中使用的许多指令（例如 attr 和 include）只不过是类 Module 的私有实例方法。

#### 22.9.1 从类中创建对象

```ruby
obj = classexpr.new [([args, ...])]
```

这是通过调用 `classexpr.allocate` 来完成的。你可以重载此方法，但是你的实现必须返回正确的类的对象。然后它调用新创建的对象的 `initialize`，并将传递给 `new` 的参数传递给 `initialize`。

如果类定义中重载了类方法 new，并且 new 没有调用 super，那么将无法创建该类的对象，并且调用 new 将返回 nil。

和其他方法一样，`initialize` 应当调用 super，以保证父类被适当地初始化。如果父类是 Object，则不需要这么做，因为 Object 类不做任何实例相关的初始化。

#### 22.9.2 类属性声明

类属性声明不是 Ruby 语法的一部分：它们不过是定义在类 Module 中的方法，该方法会自动创建访问类属性的方法。

### 22.10 模块定义

模块基本上是一个不能实例化的类。和类一样，在定义过程中模块体将被执行，生成的 Module 对象被存储在一个常量中。模块中可以含有类方法和实例方法，也可以定义常量和类变量。和类一样，通过使用 Module 对象作为接收者调用模块方法，通过使用 `::` 域作用符来访问常量。在模块定义中的名字可以以封装它的类或者模块做前缀。

```ruby
CONST = "outer"
module Mod
  CONST = 1
  def Mod.method1	# module method
    CONST + 1
  end
end
module Mod::Inner
  def (Mod::Inner).method2
    CONST + " scope"
  end
end

Mod::CONST			# ->	1
Mod.method1			# ->	2
Mod::Inner:method2	# ->	"outer scope"
```

#### 22.10.1 Mixins——混入模块

```ruby
class | module name
  include expr
end
```

使用 `include` 方法可以将一个模块包含到另一个模块或者类的定义中。含有 `include` 的模块或类定义可以访问它所包含模块的常量、类变量和实例方法。

如果一个模块被包括到一个类定义中，那么模块的常量、类变量和实例方法实际上被绑定到该类的一个匿名（且不可访问）超类中。类的对象会响应发送给模块实例方法的消息。

模块可以定义一个 `initialize` 方法，当创建包括模块的类的对象时，如果满足下面条件之一，那么该方法将被调用：

*   类没有定义它自己的 `initialize` 方法
*   类的 `initialize` 方法调用了 `super`

模块也可以在顶层被包括，在这种情况下，顶层可以访问模块的常量、类变量和实例方法。

##### 模块函数

实例方法定义的功能不能通过模块方法来实现。

方法 `Module#module_function` 通过拷贝一个或多个模块实例方法的定义来创建相应的模块方法来解决这个问题。

```ruby
module Math
  def sin(x)
	# xxx
  end
  module_function :sin
end
Math.sin(1)
include Math
sin(1)
```

实例方法和模块方法是两个不同的方法：方法定义被 `module_function` 拷贝出来而不是建立别名。

### 22.11 访问控制

Ruby 为模块和类的常量和方法定义了 3 种保护级别：

*   Public
*   Protected
*   Private

### 22.12 Blocks、Closures 和 Proc 对象

```ruby
invocation do |a1, a2, ... |
end
invocation { |a1, a2, ... }
```

花括号具有高优先级，而 do 的优先级较低。如果方法调用有不被括在括号内的参数，那么花括号形式的 block 将会绑定到最后一个参数，而不是整个调用上。do 形式会绑定到调用上。

block 是 closure；它能记住其被定义时的上下文，并在被调用时使用该上下文。上下文中包含 self 的值、常量、类变量、局部变量和任意被截获的 block。

#### 22.12.1 Proc 对象，break 和 next

Block 不是对象，但是它们能被转换成类 Proc 的对象。有 3 种方式将 block 转换成 Proc 对象。

*   通过传递 block 给一个方法，该方法的最后一个参数前有一个地址符 &。该参数会接受 block 作为 Proc 对象。
*   通过调用 `Porc.new`，再将它和 block 关联。

```ruby
block = Proc.new { "a block" }
```

*   通过调用方法 `Kernel.lambda`（或者等价的、有点过时的方法 `Kernel.proc`）关联 block 到方法调用

```ruby
block = lambda { "a block" }
```

前两种风格的 Proc 对象在使用时是相同的。我们称这些对象为 `raw procs`。第三种风格，由 lambda 生成，为 Proc 对象添加了一些额外的功能，称这种对象为 lambdas。

无论在哪种 block 内，next 语句将退出 block。block 的值是传递给 next 的值，如果没有值传递给 next，则为 nil。

```ruby
def meth
  res = yield
  "The block returns #{res}"
end
meth { next 99 }	# ->	"The block returns 99"

pr = Proc.new { next 99 }
pr.call				# ->	99

pr = lambda { next 99 }
pr.call				# ->	99
```

在 `raw proc` 中，break 语句可以终止调用 block 的方法。方法的返回值为传递给 break 的参数。

#### 22.12.2 Return 和 Blocks

处于作用域 block 内的 return 和该作用域的 return 作用一样。block 内原上下文不再有效的 return 会引发异常（因上下文不同会引发 LocalJumpError 或 ThreadError 异常）。

如果使用 `Proc.new` 从 block 中创建一个 proc 对象，该 proc 行为类似于 block，则前面的规则也适用。

如果 Proc 对象是使用 `Kernel.proc` 或 `Kernel.lambda` 来创建的，则它的行为更像自立的方法体：`return` 会从 block 中返回到 block 的调用者。

```ruby
def meth5
  p = lambda { return 99 }
  res = p.call
  "The block returned #{res}"
end

meth5	# ->	"The block returned 99"
```

### 22.13 异常

Ruby 异常是类 Exception 或其子类的对象。

#### 22.13.1 引发异常

`Kernel.raise` 方法可以引发一个异常。

```ruby
raise 
raise string
raise thing [, string [stack trace]]
```

第一种形式重新引发 `$!` 中存储异常，如果 `$!` 是 `nil`，则引发一个新的 `RuntimeError`

第二种形式创建一个新的 RuntimeError 异常，并设置其消息为给定的字符串。

第三种形式通过在其第一个参数上调用 exception 方法创建一个异常对象，然后设置异常消息和调用栈为第二个和第三个参数。

类 Exception 和其对象含有一个称为 exception 的工厂方法，所有异常类的名字和实例可以用作 raise 的第一个参数。

当异常发生时，Ruby 会将异常对象的引用存储在全局变量 `$!` 中。

#### 22.13.2 处理异常

在如下地方可以处理异常：

*   在 `begin/end` block 的作用域内

```ruby
begin
  # code
rescue 
  # xxx
else
  # xxx
ensure
  # yyy
end
```

*   在方法体内

```ruby
def method and args
  # code
rescue
  # yyy
else
  # zzz
ensure
  # xxx
end
```

*   以及一个可执行语句的后面

```ruby
statement [rescue statement]
```

不带参数的 rescue 语句被当做它好像带一个 StandardError 参数，这意味着一些底层的异常无法被不带参数的 rescue 类捕获。

Ruby 依次比较当前被引发的异常和 rescue 的每个参数；每个参数被 `parameter === $1` 测试。如果匹配的 rescue 语句以 `=>` 和一个变量名结束，那么该变量将被设置为 `$!`

在 rescue 语句中，不带参数的 raise 将重新引发 `$!` 异常。

一条语句可以含有一个可选的 rescue 修饰符，该修饰符后跟另一条语句（还可以跟另一个 rescue 修饰符等）

如果异常在 rescue 修饰符的左边发生，那么左边的语句将被放弃，而且整行的值为右边语句的值。

```ruby
values = ["1", "2.3", /pattern/]
result = values.map { |v| Integer(v) rescue Float(v) rescue String(v) }
result	# ->	[1, 2.3, "(?-mix:pattern)"]
```

retry 语句可以用在 rescue 语句中以重新运行 `begin/end` block。

### 22.14 Catch 和 Throw

方法 `Kernel.catch` 将执行与之相关联的 block。

```ruby
catch (symbol|string) do
  # block...
end
```

方法 `Kernel.throw` 将中断语句的正常执行。

```ruby
throw(symbol|string[, obj])
```

当 throw 执行时，Ruby 在调用栈中向上搜索直到找到匹配符号或字符串的第一个 catch block。

如果 throw 的第二个参数存在，那么它的值将作为 catch 的值返回。当搜索对应的 catch 时，Ruby 检查它遇到的任何 block 表达式的 ensure 语句。

如果没有匹配 throw 的 catch block，Ruby 将会在 throw 的位置引发一个 NameError 异常。
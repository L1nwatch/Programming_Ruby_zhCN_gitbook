### 5.1 数字

Ruby 支持整数和浮点数。整数可以是任何长度（其最大值取决于系统可用内存的大小）。一定范围内的整数（通常是 `-2^30 ~ 2^30 - 1` 或 `-2^62 ~ 2^62 - 1` 在内部以二进制形式存储，它们是 Fixnum 类的对象。这个范围之外的整数存储在 `Bignum` 类的对象中（目前实现为一个可变长度的短整型集合）。这个处理是透明的，Ruby 会自动管理它们之间的来回转换。

可通过 `num.class` 来查看

在书写整数时，可以使用一个可选的前导符号，可选的进制指示符（0 表示八进制，0d 表示十进制【默认】，0x 表示十六进制，0b 表示二进制），后面跟一串符合适当进制的数字。下画线在数字串中被忽略（一些人使用它们来代替逗号）

```ruby
0d123456	# =>	123456
123_456		# =>	123456
0xaabb		# =>	43707
0377		# =>	255
-0b10_1010	# =>	-42
```

控制字符的整数值可以使用 `?\C-x` 和 `?\cx`（x 的 control 版本（Ctrl + X 的值），是 `x & 0x9f`）生成。元字符（`x | 0x80`）可以使用 `?\M-x` 生成（Alt 键）。元字符和控制字符的组合可以使用 `?\M-\C-x` 生成。可以使用 `?\\` 序列得到反斜线字符的整数值。

```ruby
?a			# =>	97	# ASCII character
?\n			# =>	10	# code for a newline(0x0a)
?\C-a		# =>	1	# control a = ?A & 0x9f = 0x01
?\M-a		# =>	225	# meta sets bit 7
?\M-\C-a	# =>	129	# meta and control a
?\C-?		# =>	127 # delete character
```

与原生体系结构的 double 数据类型相对应，带有小数点和`/`或幂的数字字面量被转换成浮点对象。你必须在小数点之前和之后都给出数字（如果把 `1.0e3` 写成 `1.e3`，Ruby 会试图调用 `Fixnum` 类的 e3 方法）。

所有数字都是对象，并且可以对各种形式的消息作出响应。

整数也支持几种有用的迭代器，比如 `times`、`upto`、`downto`，它们在两个整数之间分别向上和向下迭代。另外 Numeric 类提供了更通用的 step 方法，它更像传统的 for 循环。

```ruby
3.times	{ print "X "}				# =>	X X X
1.upto(5)	{|i| print i, " "}		# =>	1 2 3 4 5
99.downto(95) {|i| print i, " "}	# =>	99 98 97 96 95
50.step(80, 5) {|i| print i, " "}	# =>	50 55 60 65 70 75 80
```

那些只包含数字的字符串，当在表达式中使用时，不会被自动转换成数字：

```ruby
some_file.each do | line |
  v1, v2 = line.split
  print Integer(v1) + Integer(v2), " "
end
```

### 5.2 字符串

双引号字符串支持更多的转义序列。如果代码只是全局变量、类变量或实例变量的话，花括号可以忽略。

要进行插入替换的代码可以是一条或多条语句，而不仅仅是一个表达式：

```ruby
puts "now is # {def the(a)
					'the ' + a
				end
				the('time')
				}for all good coders..."
```

另外还有 3 种方式去构建字符串字面量：`%q`、`%Q`、`here documents`

`%q`、`%Q` 分别开始界定单引号和双引号的字符串（可以把 `%q` 看成薄的引号`'`，把 `%Q` 看成厚的引号`"`）。

```ruby
%q/general single-quoted string/	# ->	general single-quoted string
%Q!general doubke-quoted string!	# ->	general doubke-quoted string
%Q{Seconds/day: #{24 * 60 * 60}}	# ->	Seconds/day: 86400
```

跟在 q 或 Q 后面的字符是分界符。如果它是开始（opening）的方括号 `"["`，花括号 `"{"`，括号 `"{"`，或小于号 `"<"`，字符串被一直读取直到发现相匹配的结束符号。否则，字符串会被一直读取，直到出现下一个相同的分界符。分界符可以是任何一个非字母数字的单字节字符。

最后，可以使用 `here document` 构建字符串：

```ruby
string = <<END_OF_STRING
The body of the string
text that followed the '<<'
END_OF_STRING
```

`here document` 由源文件中那些行但没有包含在 `<<` 字符后面指明终结字符串的行组成。一般情况下，终结符（terminator）必须在第一列出现。当然，如果把一个减号放在 `<<` 字符后面，就可以缩进编排终结符。

```ruby
print <<-STRING1, <<-STRING2
Concat
String1
  encate
  STRING2
```

注意，前导空格不会删去

#### 5.2.1 操作字符串

`chmop` 可用于去除换行符

`split` 可实现分割：

```ruby
line.chmop.split(/\s*\|\s*/)
```

有许多方式可以删除多余空格，但是最简单的方式可能是 `String#squeeze`，它修建（trim）重复字符：

```ruby
name.squuze!(" ")
```

`String#scan` 类似于 `split`，因为它根据模式（pattern）把字符串分成几块。但是与 split 不一样，使用 scan 可以指定希望这些块去匹配的模式。

```ruby
mins, secs = length.split(/:/)
# 等价
mins, secs = length.scan(/\d+/)
```

### 5.3 区间

Ruby 使用区间去实现 3 种不同的特性：序列（sequences）、条件（conditionals）和间隔（intervals）

#### 5.3.1 区间作为序列

在 Ruby 中，使用 `..` 和 `...` 区间操作符来创建序列。两个点的形式是创建闭合的区间（包括右端的值），而 3 个点的形式是创建半闭半开的区间（不包括右端的值）。

在 Ruby 中，区间没有在内部用列表（list）表示：`1..100000` 序列被存储为 Range 对象，它包含对两个 Fixnum 对象的引用。如果需要，可以使用 `to_a` 方法把区间转换成列表。

Ruby 可以根据你所定义的对象来创建区间。唯一的限制是这些对象必须返回在序列中的下一个对象作为对 succ 的响应，而且这些对象必须是可以使用 `<=>` 来比较的。

以下的 VU 类实现了 succ  和 `<=>` 方法，因此它可以作为区间：

```ruby
class VU
  include Comparable
  attr :volume
  def initialize(volume)	# 0..9
    @volume = volume
  end
  def inspect
    '#' * @volume
  end
  def <=>(other)
    self.volume <=> other.volume
  end
  def succ
    raise(IndexError, "Volume too big") if @volume >= 9
    VU.new(@volume.succ)
  end
end

medium_volume = VU.new(4)..VU.new(7)
medium_volume.to_a					# ->	[####, #####, ######, #######]
medium_volume.include?(VU.new(3))	# ->	false
```

#### 5.3.2 区间作为条件

在这里它们表现得就像某种双向开关——当区间第一部分的条件为 true 时，它们就打开，当区间第二部分的条件为 true 时，它们就关闭。例如，下面的代码段，打印从标准输入得到的行的集合，每组的第一行包含 start 这个词，最后一行包含 end 这个词。

```ruby
while line = gets
  puts line if line =~ /start/ .. line =~ /end/
end
```

早期的 Ruby 版本中，裸区间（bare range）可以在 `if`、`while` 和类似的语句中作为条件来使用。比如，可能会把先前的代码写成：

```ruby
while gets
  print if /start/../end/
end
```

这种写法不再支持。但不幸的是，它不会引发任何错误；且每次测试都会成功。

#### 5.3.3 区间作为间隔

间隔测试：看看一些值是否会落入区间表达的间隔内。使用 `===` 即 `case equality` 操作符可以做到这一点：

```ruby
(1..10)	===	5		# ->	true
(1..10)	===	15		# ->	false
(1..10)	===	3.14	# ->	true
('a'..'j')	===	'c'	# ->	true
('a'..'j')	=== 'z'	# ->	false
```

### 5.4 正则表达式

正则表达式是 Regexp 类型的对象。可以通过显示地调用构造函数或使用字面量形式 `/pattern/` 和 `%r{pattern}` 来创建它们。

```ruby
a = Regexp.new('^\s*[a-z]')	# ->	/^\s*[a-z]/
b = /^\s*[a-z]/				# ->	/^\s*[a-z]/
c = %r{^\s*[a-z]}			# ->	/^\s*[a-z]/
```

一旦有了正则表达式对象，可以使用 `Regexp#match(string)` 或匹配操作符 `=~`（肯定匹配）和 `!~`（否定匹配）对字符串进行匹配。匹配操作符对 String 和 Regexp 对象均有定义。匹配操作符至少有一个操作数必须为正则表达式。（早期版本的 Ruby 当中，这两个操作数可能都是字符串，且第二个操作数会暗中被转换成正则表达式。）

```ruby
name = "Fats Waller"
name =~ /a/	# -> 1
name =~ /z/	# -> nil
/a/ =~ name	# -> 1
```

匹配操作符返回匹配发生的字符位置。它们也有副作用，会设置一些 Ruby 变量。`$&` 得到与模式匹配的那部分字符串，$\` 得到匹配之前的那部分字符串，而 `$'` 得到匹配之后的那部分字符串：

```ruby
def show_regexp(a, re)
  if a =~ re
    "#{$`}<<#{$&}>>#{$'}"
  else
    "no match"
  end
end
show_regexp("very intersting", /t/)	# -> very in<<t>>eresting
```

这个匹配也设置了线程局部变量（`thread-local variables`），`$~` 与 `$1` 直到 `$9`。`$~` 变量是 MatchData 对象，它持有你想知道的有关匹配的所有信息。`$1` 等持有匹配各个部分的值。

#### 5.4.1 模式

##### 锚点

^ 和 & 模式分别匹配行首和行尾。它们常常被用来锚定（author）模式匹配。`\A` 序列匹配字符串的开始，而 `\z` 和 `\Z` 匹配字符串的结尾（实际上，除非字符串以 `\n` 结束，`\Z` 才会匹配字符串的结尾，它会在 `\n` 之前匹配）。

```ruby
show_regexp("this is\nthe time", /^the/)		# -> this is\n<<the>> time
show_regexp("this is\nthe time", /is$/)			# -> this <<is>>\nthe time
show_regexp("this is\nthe time", /\Athe/)		# -> <<this>> is\nthe time
show_regexp("this is\nthe time", /\Athe/)		# -> no match
```

同样的，`\b` 和 `\B` 模式分别匹配词的边界和非词（nonword）的边界。组词字符可以是字母、数字和下画线。

```ruby
show_regexp("this is\nthe time", /\bis/)		# -> this <<is>>\nthe time
show_regexp("this is\nthe time", /\Bis/)		# -> th<<is>> is\nthe time
```

##### 字符类

特殊正则表达式字符的意义——`.|()[{+^$*?` 在方括号里面是关闭的。不过，正常的字符替换仍然发生，因此 `\b` 表示回退空格，而 `\n` 表示回车换行符。

| 序列   | 是             | 含义           |
| ---- | ------------- | ------------ |
| \d   | [0-9]         | 数字字符         |
| \D   | [^0-9]        | 除数字之外的任何字符   |
| \s   | [ \t\r\n\f]   | 空格字符         |
| \S   | [^ \t\r\n\f]  | 除空格之外的任何字符   |
| \w   | [A-Za-z9-9_]  | 组词字符         |
| \W   | [^A-Za-z0-9_] | 除组词字符之外的任何字符 |

| POSIX 字符类  | 含义                      |
| ---------- | ----------------------- |
| [:alnum:]  | 字母和数字                   |
| [:alpha:]  | 大写或小写字母                 |
| [:blank:]  | 空格和制表符                  |
| [:cntrl:]  | 控制字符（至少 0x00-0x1f，0x7f） |
| [:digit:]  | 数字                      |
| [:Graph:]  | 除了空格的可打印字符              |
| [:lower:]  | 小写字符                    |
| [:print:]  | 任何可打印字符（包括空格）           |
| [:punct:]  | 除了空格和字母数字的可打印字符         |
| [:space:]  | 空格（等同于\s）               |
| [:upper:]  | 大写字母                    |
| [:xdigit:] | 16 进制数字（0-9，a-f，A-F）    |

```ruby
show_regexp("Price $12.", /[aeiou]/)			# ->	Pr<<i>>ce $12.
show_regexp("Price $12.", /[\s]/)				# ->	Price<< >>$12.
show_regexp("Price $12.", /[[:digit:]]/)		# ->	Price $<<1>>2.
show_regexp("Price $12.", /[[:space:]]/)		# ->	Price<< >>$12.
show_regexp("Price $12.", /[[:punct:]aeiou]/)	# ->	Pr<<i>>ce $12.
```

在方括号内，`c1-c2` 序列表达 `c1` 和 `c2` 之间的所有字符，并包含 `c2`

如果想在字符类包含字面量字符`]`和`-`，它们必须出现在开始处。把 `^` 直接放在开始的方括号后面会对字符类求反。

```ruby
a = 'see [Design Patterns-page 123]'
show_regexp(a, /[]]/)		# ->	see [Design Patterns-page 123<<]>>
show_regexp(a, /[-]/)		# ->	see [Design Patterns<<->>page 123]
show_regexp(a, /[^a-z]/)	# ->	see<< >>[Design Patterns-page 123<<]>>
show_regexp(a, /[^a-z\s]/)	# ->	see <<[>>Design Patterns-page 123<<]>>
```

出现在方括号外面的点号（.）表示除回车换行符之外的任何字符（尽管在多行模式下它会匹配回车换行符）。

##### 重复

`r{m,}`：匹配至少 “m” 次 r 的出现

重复构造有高的优先级——它们只绑定到模式内先前的正则表达式。必须小心使用 \* 构造。

贪心（greedy）模式，默认情况下它们会匹配尽可能多的字符串。可以通过添加问号后缀让它们匹配最少的字符串来改变这个默认行为。

##### 替换

未转义的竖线（|）要么是匹配在它之前的正则表达式，要么是匹配在它之后的正则表达式

然而竖线的优先级非常低，有时需要使用编组（grouping）来重载默认的优先级，比如以下例子匹配 red ball 或 angry sky，但不会匹配 red ball sky 或 red angry sky：

```ruby
show_regexp(a, /red ball|angry sky/)	# ->	<<red ball>> blue sky
```

##### 编组

可以使用括号在正则表达式中编组（group）词目。组内的所有东西被当作单个正则表达式对待。

括号也收集模式匹配的结果。Ruby 计算开始括号的数目，保存每个开始括号和相应的关闭括号之间部分匹配的结果。可以在模式的剩余部分和 Ruby 程序中使用这种部分匹配。在模式内部，`\1` 序列指的是第一个组的匹配，`\2` 序列指的是第二个组的匹配。在模式外面，特殊变量 `$1` 和 `$2` 等起到相同作用。

在匹配的剩余部分使用当前部分匹配的能力，能够让你在字符串中寻找各种形式的重复：

```ruby
show_regexp('He said "Hello"', /(\w)\1/)	# ->	He said "He<<ll>>o"
show_regexp('Mississippi', /(\w+)\1/)		# ->	M<<ississ>>ippi
```

也可以使用向后引用去匹配分界符：

```ruby
show_regexp('He said "Hello"', /(["']).*?\1/)	# ->	He said <<"Hello">>
show_regexp("He said 'Hello'", /(["']).*?\1/)	# ->	He said <<'Hello'>>
```

#### 5.4.2 基于模式的替换

sub 和 gsub 的第二个参数可以是 String 或 block。如果使用 block，匹配的子字符串会被传递给 block，同时 block 的结果值会被替换到原先的字符串中：

```ruby
a = "the quick brown fox"
a.sub(/^./) {|match| match.upcase }			# ->	"The quick brown fox"
a.gsub(/[aeiou]/) {|vowel| vowel.upcase}	# ->	"thE qUIck brOwn fOx"
```

匹配词的首字符的模式是 `\b\w`——它寻找后面跟着词字符（word character）的词边界：

```ruby
def mixed_case(name)
  name.gsub(/\b\w/) {|first| first.upcase}
end

mixed_case("fats waller")	# ->	"Fats Waller"
```

##### 替换中的反斜线序列

序列 `\1` 和 `\2` 等在模式中可用，它们代表至今为止第 n 个已匹配的组。相同的序列也可以作为 sub 和 gsub 的第二个参数。

其他的反斜线序列在替换字符串中起的作用：`\&`（最后的匹配）、`\+`（最后匹配的组）、\\\`（匹配之前的字符串），`\'` 匹配之后的字符串和 `\\`（字面量反斜线）

可以利用 `\&` 表示反斜线替换：

```ruby
str = 'a\b\c'
str.gsub(/\\/, '\\\\\\\\')	# ->	"a\\b\\c"
str = 'a\b\c'
str.gsub(/\\/, '\&\&')		# ->	"a\\b\\c"
```

如果使用 gsub 的 block 形式，用来替换的字符串会被且仅被分析一次（在语法分析阶段）：

```ruby
str = 'a\b\c'
str.gsub(/\\/) { '\\\\' }	# ->	"a\\b\\c"
```

#### 5.4.3 面向对象的正则表达式

在 Matz 设计 Ruby 时，他设计了一个完全面向对象的正则表达式处理系统。然后在它之上包装（wrap）所有这些 `$` 变量，使它们看起来对 Perl 程序员很熟悉。对象和类还在这里，只不过就在表面下。

`Regexp#match` 方法对字符串匹配正则表达式。如果失败了，它返回 `nil`。如果成功，则返回 `MatchData` 类的一个实例。`MatchData` 对象让你访问关于这次匹配的所有可用信息。所有这些好东西是通过 `$` 变量得到的。它们绑定在一个小巧方便的对象里。

```ruby
re = /(\d+):(\d+)/	# match a time hh:mm
md = re.match("Time: 12:34am")
md.class		# 	->	MatchData
md[0]			# == $&		->	"12:34"
md[1]			# == $1		->	"12"
md[2]			# == $2		->	"34"
md.pre_match	# == $`		->	"Time: "
md.post_match	# == $'		->	"am"
```

匹配数据存储在它自己的对象里，这样可以同时保留两个或多个模式匹配的结果，这些事情使用那些 `$` 变量是做不出来的。

每次模式匹配发生后，Ruby 把对结果（`nil` 或一个 `MatchData` 对象）的引用保存到线程局部变量中（可通过 `$~` 访问）。然后所有其他的正则表达式变量都是从这个对象派生出来的。

以下说明了所有其他与 MatchData 相关的那些 `$` 变量真的是从 `$~` 中得到的：

```ruby
re = /(\d+):(\d+)/
md1 = re.match("Time: 12:34am")
md2 = re.match("Time: 10:30pm")
[$1, $2]	# last successful match			->	["10", "30"]
$~ = md1
[$1, $2]	# previous successfule match	->	["12", "34"]
```
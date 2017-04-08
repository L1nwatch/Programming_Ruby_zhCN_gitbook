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

#### 5.3.1 区间作为序列


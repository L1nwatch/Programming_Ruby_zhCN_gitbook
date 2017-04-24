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


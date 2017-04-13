Ruby 提供了两套 I/O 例程（routine）。第一套接口是由 Kernel 模块实现了一整套 I/O 相关的方法：`gets`、`open`、`print`、`printf`、`putc`、`puts`、`readline`、`readlines` 和 `test`

第二种方式是使用 IO 对象，这种方式可以对 IO 进行更多的控制。

### 10.1 什么是 IO 对象

Ruby 定义了一个 IO 基类来处理输入和输出。类 File 和 BasicSocket 都是该基类的子类。IO 对象是 Ruby 程序和某些外部资源之间的一个双向通道。

### 10.2 文件打开和关闭

当创建一个文件时，你还可以指定文件的许可权限。

方法 `File.open` 也可以打开文件，通常情况下它和 `File.new` 行为相似，但是当和 block 一起调用时就有区别了，当 block 退出时，文件会被自动关闭。

以 block 调用 `File.open` 的形式不用担心发生异常之后 `file.close` 不会被调用的问题。如果 block 内引发了异常，在异常传递给调用者之前文件会被关闭。

### 10.3 文件读写

用于 "简单" I/O 的所有方法都适用于文件对象。`gets` 从标准输入读取一行（或者从执行脚本的命令行上指定的任意文件），而 `file.gets` 从文件对象 `file` 中读取一行。

#### 10.3.1 读取迭代器

`IO#each_byte` 将以从 IO 对象中获得的下一个 8 位字节为参数，调用相关联的 block。

```ruby
File.open("testfile") do |file|
  file.each_byte {|ch| putc ch; print '.'}
end
```

`IO#each_line` 以文件的每一行为参数调用相关联的 block。可以使用 `String#dump` 来显示换行符。还可以将任意字符序列传递给 `each_line` 作为行分隔符，然后它会根据该字符序列相应地分隔输入字符串，并返回分割后的每一行。

`IO.foreach` 方法以 I/O 数据源的名字作为参数，以读模式打开它，并以文件中的每一行为参数调用关联的迭代器，最后会自动关闭该文件。

```ruby
IO.foreach("testfile") {|line| puts line}
```

#### 10.3.2 写文件

除了少数几个例外，你传递给 `puts` 和 `prints` 的任意对象都会被该对象的 `to_s` 方法转换成一个字符串。如果由于某种原因，`to_s` 方法未能返回一个合法的字符串，含有对象类名和 `ID` 的一个字符串就会被创建，类似于 `#<ClassName:0x123456>`

`nil` 对象会输出字符串 `nil`，而传递给 `puts` 的数组会依次将它的元素打印出来。

如果你想写入二进制数据，可以调用 `IO#print`，并以包括待写入字节的字符串作为参数。你也可以使用底层的输入输出例程（`IO#sysread` 和 `IO#syswrite`）

要将二进制数据存储到字符串中，常用的有 3 个方法：字符串的字面量、一个字节一个字节地存入、或使用 `Array#pack`

```ruby
str1 = "\001\002\003"	# ->	"\001\002\003"
str2 = ""
str2 << 1 << 2 << 3		# ->	"\001\002\003"
[1, 2, 3].pack("c*")	# ->	"\001\002\003"
```

你也可以添加对象到 IO 输出流：

```ruby
endl = "\n"
STDOUT << 99 << " red balloons" << endl
```

方法 `<<` 使用 `to_s` 将它的参数转换成字符串，然后再按照它的方式传递。

##### 使用字符串 I/O

`StringIO` 对象，它们读写的是字符串而不是文件。

```ruby
require "stringio"

ip = StringIO.new("now is\nthe time\n to learn\nRuby!")
op = StringIO.new("", "w")

ip.each_line do |line|
  op.puts line.reverse
end
```

### 10.4 谈谈网络

Ruby 善于处理网络协议，无论是底层协议还是高层协议。

Ruby 的套接字库提供了一组类，这些类让你可以访问 TCP、UDP、SOCK、Unix 域套接字，以及在你的体系结构上支持的任意其他套接字类型。

```ruby
require "socket"
client = TCPSocket.open("127.0.0.1", "finger")
client.send("mysql\n", 0)
puts client.readlines
client.close
```

在较高的层次上，`lib/net` 提供了一组模块来处理应用层协议（目前支持 FTP、HTTP、POP、SMTP 和 telnet）

```ruby
require "net/http"
h = Net::HTTP.new("www.pragmaticprogrammer.com", 80)
response = h.get("/index.html", nil)
if response.message == "OK"
  puts response.body.scan(/<img src="(.*?)"/m).uniq
end
```

还可以在更高层进行处理。通过装载 `open-uri` 库到程序中，方法 `Kernel.open` 立刻就可以识别文件名中的 `http://` 和 `ftp://` 等 URL。

```ruby
require "open-uri"
open("http://xxx.com") do |f|
  puts f.read.scan(/<img src="(.*?)"/m).uniq
end
```
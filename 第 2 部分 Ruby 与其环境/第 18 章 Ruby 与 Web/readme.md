先从运用 Ruby 程序作为通用网关接口（Common Gateway Interface, CGI）程序开始：

### 18.1 编写 CGI 脚本

使用 Ruby 编写 CGI 脚本非常容易。要让 Ruby 脚本产生 HTML 输出：

```ruby
#!/usr/bin/ruby
print "Content-type: text/html\r\n\r\n"
print "<html><body>Hello #{Time.now}!</body></html>\r\n"
```

将这个脚本放到 CGI 目录中，使其有可执行的权限，然后你可以通过浏览器来访问它。

不过这种编程方式太底层了。

#### 18.1.1 使用 `cgi.rb`

类 CGI 提供了编写 CGI 脚本的支持。使用它，你可以操作表单、cookie 和环境；维护有状态的会话等。是一个相当大的类。

#### 18.1.2 引用

当要处理 URL 和 HTML 编码时，可以使用 CGI 提供的 `CGI.escape` 和 `CGI.unescape` 两个例程（routine）。

```ruby
require "cgi"
puts CGI.escape("AAAAxxxxx/ooo&xxxx")
```

更常见的，你可能想要转义 HTML 特殊字符：

```ruby
require "cgi"
puts CGI.escapeHTML("a < 100 && b > 200")
```

也可以只对字符串中某个特定的 HTML 元素进行转义：

```ruby
require "cgi"
puts CGI.escapeElement("<ht><a href='/mp3'>Click</a><br>", "A")
```

这里只对 A（Auchor，锚点）元素进行转义，而其他元素则保持原样。这些方法都有反向的版本（`un-`）来恢复原来的字符串。

#### 18.1.3 查询参数

从浏览器到你应用的 HTTP 请求中可能包括参数，或者作为 URL 的一部分，或者将数据嵌入到请求体中。

类 CGI 为你提供了几种方式来访问表单数据。首先，可以把 CGI 对象看做是一个散列表，通过字段名（field）索引并返回字段的值：

```ruby
require "cgi"
cgi = CGI.new
cgi["name"]		# ->	"Dave Thomas"
cgi["reason"]	# ->	"fiexible"
```

我们可以通过使用 `CGI#params` 方法来要求全部查看。`params` 返回的值类似一个散列表，保存有请求的参数。你可以读写这个散列表。

```ruby
cgi.params["reason"]
```

你可以使用 `CGI#has_key?` 来判断请求中是否包括某个特定参数：

```ruby
cgi.has_key?("name")	# =>	true
```

##### 生成 HTML

CGI 包括大量的方法用来创建 HTML——每个元素一个方法。为了让元素嵌套更容易些，这些方法将代码 block 作为其内容。这些代码 block 应该返回一个 String 作为该元素的内容：

```ruby
reqire "cgi"
cgi = CGI.new("html3")	# add HTML generation methods
cgi.out {
  cgi.html {
    cgi.head { "\n" + cgi.title{"This is a Test"}} +
      cgi.body { "\n" +
          cgi.form { "\n" +
            cgi.hr +
              cgi.h1 { "A Form: " } + "\n" + 
              cgi.textarea("get_text") + "\n" +
              cgi.br +
              cgi.submit
            }
      }
  }
}
```

当提交返回后，你讲得到一个名为 `get_text` 的 CGI 参数，其中包括用户所输入的文本。

多数人倾向直接书写 HTML，使用模板系统，或者使用应用框架（Iowa）。

##### 模板系统

模板系统可以让你将应用的表示和逻辑分离开来。RubyGarden 的 wiki 列出了多，现在只看其中的 3 个：RDoc 模板、Amrita 和 `erb/eruby`

###### RDoc 模板

RDoc 文档系统含有一个非常简单的模板系统，用来将它的 XML 文档产生 HTML 输出。因为 RDoc 是作为标准 Ruby 的一部分而分发的，安装了 Ruby1.8.2 之后就可以获得这个模板系统。不过，这个模板系统并不使用传统的 HTML 或 XML 标记，因此有 RDoc 模板标记的文件，可能不容易用传统的 HTML 编辑工具来编辑。

```ruby
require "rdoc/template"
HTML = %{Hello, %name%.
<p>
The reasons you gave were:
<ul>
START:reasons
<li>%reason_name% (%rank%)
END:reasons
</ul>
}
data = {
  "name" => "Dave Thomas",
  "reasons" => [
    { "reason_name" => "fiexible", "rank" => "87"},
    { "reason_name" => "transparent", "rank" => "76"},
    { "reason_name" => "fun", "rank" => "94"},
  ]
}
t = TemplatePage.new(HTML)
t.write_html_on(STDOUT, data)
```

向构造函数传入一个包含了要应用的模板的字符串。然后将一个包括名字和值的散列表传入 `write_html_on` 方法。如果模板包括 `%xxx%` 序列，则查找散列表，而对应名字 xxx 的值会被替换进去。如果模板包括 `START:yyy`，对应 yyy 的散列值被认为是一个散列数组。在 `START:yyy` 和 `END:yyy` 之间的模板行重复应用于数组的每个元素。模板还支持条件语句：`IF:zzz` 和 `ENDIF:zzz`。

###### Amrita

Amrita 是生成 HTML 文档的库，其模板本身也是有效的 HTML。

Amrita 使用 HTML 元素的 id 标记来判断值是否要替换。如果对应给定名字的值是 `nil` 或 `false`，HTML 元素不会包含到输出结果中。如果只是一个数组，它迭代对应的 HTML 元素。

```ruby
require "amrita/template"
include Amrita
HTML = %{<p id="greeting" />
<p>The reasons you gave were:</p>
<ul>
<li id="reasons"><span id="reason_name"></span>,
<span id="rank"></span>
</ul>
}
data = {
  :greeting => "Hello, Dave Thomas",
  :reasons => [
    { :reason_name => "fiexible", :rank => "87"},
    { :reason_name => "transparent", :rank => "76"},
    { :reason_name => "fun", :rank => "94"},
  ]
}
t = TemplateText.new(HTML)
t.prettyprint = true
t.expand(STDOUT, data)
```

###### erb 和 eruby

我们可以将 Ruby 嵌入到 HTML 文档中。

eRuby 存在很多不同的实现，包括 `erb` 和 `eruby`。

在 HTML 中嵌入 Ruby 是非常强大的概念——它基本上提供了同 ASP、JSP 或 PHP 对等的工具，但是具有 Ruby 的全部能力。

**使用 erb**

erb 通常被作为一个过滤器。输入文件中的文本除了以下情况外，其他不会被更改。

| 表达式                      | 描述                       |
| ------------------------ | ------------------------ |
| `<% ruby code %>`        | 执行分隔符之间的 Ruby 代码         |
| `<%= ruby expression %>` | 对 ruby 表达式求值，并用表达式的值替换序列 |
| `<%# ruby code %>`       | 忽略分隔符之间的 Ruby 代码（对测试有用）  |
| `% line of ruby code`    | 由百分号开头的文本行被认为只包括 Ruby 代码 |

你可以如下调用 `erb`：

```shell
erb [options][document]
```

如果忽略 document 参数，erubg 将从标准输入中读取。

`erb` 是通过将它的输入改写为一个 Ruby 脚本并执行该脚本来工作的。你可以使用 `-n` 或 `-x` 选项来查看 erb 生成的 Ruby 脚本。

```shell
erb -x f1.erb
```

注意 erb 是如何构造字符串 `_erbout` 的，它包括来自模板的静态字符串，以及表达式的执行结果。

#### 18.1.4 在 Apache 中安装 eruby

eruby 有较好的性能。你可以配置 Apache Web 服务器自动使用 eRuby 来解析内嵌 Ruby 的文档，和 PHP 的方式一样。你可以用 `.rhtml` 作为后缀创建内嵌 Ruby 的文件，然后配置 Web 服务器对这些文档运行 eruby 程序来产生想要的 HTML 输出。

### 18.2 Cookies

Ruby CGI 类为你处理加载和保存 cookies。你可以使用 `CGI#cookies` 方法来访问与当前请求相关联的 cookies，你可以通过将 `CGI#out` 的 `cookies` 参数设置为单个 cookie 或 cookies 数组的引用，将 cookies 返回给浏览器

```ruby
#!/usr/bin/ruby
COOKIE_NAME = "chocolate chip"
require "cgi"
cgi = CGI.new
values = cgi.cookies[COOKIE_NAME]
if values.empty?
  msg = "No visited recently"
else
  msg = "visited #{values[0]}"
end
cookie = CGI::Cookie.new(COOKIE_NAME, Time.now.to_s)
cookie.expires = Time.now + 30 * 24 * 3600 # 30 days
cgi.out("cookie" => cookie){ msg }
```

#### 18.2.1 会话

Cookies 本身还需要会话（session）：一种持久化的特定 Web 浏览器和请求之间的信息。会话是由 `CGI::Session` 处理的，它使用 cookies 但是提供了一种更高层次的抽象。

你可以选择会话的存储技术，可以保存在一般文件中，在 PStore 中，在内存中，甚至在你自定义的存储中。

```ruby
require "cgi"
require "cgi/session"
cgi = CGI.new("html3")
sess = CGI::Session.new(cgi,
  "session_key" => "rubyweb",
  "prefix" => "web-session.")
if sess["lastaccess"]
  msg = "#{sess['lastaccess']}."
else
  msg = "Nothing"
end
count = (sess["accesscount"] || 0).to_i
count += 1
msg << "<p>Number of visits: #{count}"
sess["accesscount"] = count
sess["lastaccess"] = Time.now.to_s
sess.close
cgi.out {
  cgi.html {
    cgi.body {
      msg
    }
  }
}
```

上面的示例代码使用了会话的默认存储机制：将持久化数据保存到你默认的临时目录中（参见 `Dir.tmpdir`）。文件名均以 `web-session.` 开头，同时以会话 ID 的散列值结尾。

### 18.3 提升性能

可以使用 Ruby 编写 Web 的 CGI 程序，但是默认的配置是没访问 cgi-bin 的一个页面，就启动一个新的 Ruby 程序拷贝。Apache Web 服务器通过支持可加载的模块来解决这个问题。

一般来说，这些模块是动态加载的，并成为 Web 服务器运行进程的一部分——你不必每次都衍生解释器进程来响应请求；Web 服务器便是解释器。

使用 `mod_ruby`（可以从应用归档中获得），它是 Apache 的一个模块，将一个完整的 Ruby 解释器链接到 Apache Web 服务器本身。`mod_ruby` 所带的 README 提供了如何编译及安装的细节。

在安装和配置好之后，可以运行 Ruby 脚本，也如同没有使用 `mod_ruby` 一样，唯一的差是它运行得更快些。你还可以利用 `mod_ruby` 提供额外的功能（比如 Apache 请求处理的紧密集成）

在请求之间解释器始终保持在内存中，它可能会处理多个应用的请求。这些应用的库可能会产生冲突（特别当不同的库包括同名类时）。你无法假定来自浏览器会话的请求序列都是由同一个解释器处理的——Apache 使用内部的算法来分配处理程序进程。

使用 FastCGI 协议可以解决部分问题，不止对 Ruby，对所有 CGI 类型的程序都适用。它使用了一个非常简单的代理程序，通常作为 Apache 的一个模块。当请求到达时，这个代理会将它们转发到一个特定的、始终运行的进程。结果返回给代理，然后发送回浏览器。

### 18.4 Web 服务器的选择

Ruby 1.8 和之后的版本都捆绑了 WEBrick，一个灵活的、纯 Ruby 的 HTTP 服务器工具。基本上，它是一个基于插件的框架，可以让你编写服务器来处理 HTTP 请求及响应。

```ruby
#!/usr/bin/ruby
require "webrick"
include WEBrick
s = HTTPServer.new(
  :Port =>	2000,
  :Document => File.join(Dir.pwd, "/html")
)
trap("INT") { s.shutdown }
s.start
```

HTTPServer 构造函数在端口 2000 上创建了一个新的 Web 服务器。代码设置文档目录为当前目录下的 `html/` 子目录。然后在服务器开始运行之前调用 `Kernel.trap`，使其在遇到中断信号时，可以完整地关闭。

WEBrick 并不仅限于支持这种静态内容的服务。你可以像 Java servlet 容器那样使用它。

```ruby
#!/usr/bin/ruby
require "webrick"
include WEBrick
s = HTTPServer.new( :Port => 2000 )
class HelloServlet < HTTPServlet::AbstractServlet
  def do_GET(req, res)
    res["Content-Type"] = "text/html"
    res.body = %{
<html><body>
Hello. You're calling from a #{req["User-Agent"]}
<p>
  I see parameters: #{req.query.keys.join(", ")}
</body></html>
    }
end
end
s.mount("/hello", HelloServlet)
trap("INT") { s.shutdown }
s.start
```

### 18.5 SOAP 及 Web Services

Ruby 目前也提供了 SOAP 的一种实现。它可以让你编写使用 Web Services 的服务器和客户端。这些应用可以从本地或跨网络远程访问。SOAP 应用也不知道网络另一端是用何种语言实现的，因此 SOAP 是将 Ruby 应用和其他语言编写的应用进行互联的一种便捷方式。

SOAP 基本上是一种列集（marshaling）机制，它使用 XML 在网络的两个节点间发送数据。它基本上被用来实现分布进程间的远程方法调用，RPC（remote procedure call）。SIAP 服务器发布一个或多个接口。这些接口是由数据类型以及使用这些类型的方法所定义的。然后 SOAP 客户端创建本地的代理，通过 SOAP 协议连接到服务器的接口。对代理中方法的调用，会被传递到服务器对应的接口上。服务器上方法产生的返回值通过代理传递回客户端。

以下编写一个简单的 SOAP 服务：

```ruby
class InterestCalculator
  attr_reader :call_count
  def initialize
    @call_count = 0
  end
  def compound(principal, rate, freq, years)
    @call_count += 1
    principal * (1.0 + rate/freq) ** (freq * years)
  end
end
```

接下来让对象可以通过 SOAP 服务器来访问，这将让客户端程序能够通过网络调用对象的方法。

```ruby
require "soap/rpc/standaloneServer"
require "interestcalc"
NS = "http://pragprog.com/InterestCalc"
class Server2 < SOAP::RPC::StandaloneServer
  def on_init
    calc = InterestCalculator.new
    add_method(calc, "compound", "principal", "rate", "freq", "years")
    add_method(calc, "call_count")
  end
end
svr = Server2.new("Calc", NS, "0.0.0.0", 12321)
trap("INT") { svr.shutdown }
svr.start
```

这里实现了一个独立的 SOAP 服务器。当它初始化时，这个类创建了一个 InterestCalculator 对象。然后它使用 `add_method` 将这个类实现的两种方法 compound 和 `call_count` 加入到服务器中。最后，代码创建了服务器类的一个实例并运行之。构造函数的参数是应用的名字、默认的命名空间、接口使用的地址以及端口号。

其次，我们需要编写一些客户端代码来访问这个服务器。客户端为服务器上的 InterestCalculator 服务创建一个本地代理，加入它想要使用的方法，然后调用它们。

```ruby
require "soap/rpc/driver"
proxy = SOAP::RPC::Driver.new("http://localhost:12321",
  "http://pragprog.com/InterestCalc")
proxy.add_method("compound", "principal", "rate", "freq", "years")
proxy.add_method("call_count")
puts "#{proxy.call_count}"
puts "#{proxy.compound(100, 0.06, 1, 5)}"
```

要测试它，可以在终端窗口运行服务器及客户端：

```shell
terminal1% ruby server.rb 
terminal2% ruby client.rb
```

可以注意到，当第二次运行客户端时，调用次数从 2 开始。服务器创建了单独一个 InterestCalculator 对象来服务进入的请求，且这个对象为每个请求所重用。

#### 18.5.1 SOAP 和 Google

SOAP 的真正好处是，让你可以和其他的 Web 服务交互。

在向 Google 发送查询之前，需要开发者密钥。可以使用 `add_method` 调用为 `doGoogleSearch` 方法构造一个 SOAP 代理。

不过 SOAP 支持动态地发现服务器上对象的接口。这是使用 WSDL（Web Services Description Language）来完成的。WSDL 文件是一个 XML 文档，描述了一个 Web 服务接口的类型、方法以及访问的机制。SOAP 客户端可以读取 WSDL 文件来自动创建访问服务器的接口。

最后，可以使用 Google 库（可以从 RAA 得到）来更进一步。它将 Web 服务 API 封装到一个漂亮的借口后面（消除了所有额外的参数）。

```ruby
require "google"
require "cgi"
key = File.read(File.join(ENV["HOME"], ".google_key")).chomp
google = Google::Search.new(key)
result = google.search("pragmatic")
printf "%d", result.estimatedTotalResultsCount
printf "%6f", result.searchTime
first = result.resultElements[0]
puts first.title
puts first.url
puts CGI.unescapeHTML(first.snippet)
```

### 18.6 更多信息

如果觉得 SOAP 有些复杂，可以考察使用 XML-RPC。




能够内省（introspect）是 Ruby 的优点之一，在程序内部自己检验程序的方方面面。Java 把这个特性称为反射（reflection）。

### 26.1 看看对象

使用 Ruby 的 `ObjectSpace.each_object` 就可以遍历对象。

```ruby
a = 102.7
b = 95.1
ObjectSpace.each_object(Numeric) { |x| p x }
```

Float 类为最大和最小浮点数定义了常量与 epsilon，这是区分两个浮点数的最小差距。

ObjectSpace 并不知晓这些具有立即值的对象：`Fixnum`、`Symbol`、`true`、`false` 和 `nil`

#### 26.1.1 窥入对象

在静态语言中变量类型决定了它的类以及它所支持的方法。与静态语言不一样，Ruby 支持自由的（liberated）对象。

```ruby
r = 1..10
list = r.methods
list.length	# ->	68
list[0..3]	# -> ["collect", "to_a", ...]
```

看看对象是否支持某个具体的方法：

```ruby
r.respond_to?("fronzen?")	# ->	true
r.respond_to?(:has_key?)	# ->	false
```

可以确定对象的类以及它的唯一对象 ID，并测试它与其他类的关系。

```ruby
num = 1
num.id						# ->	3
num.class					# ->	Fixnum
num.kind_of? Fixnum			# ->	true
num.kind_of? Numeric		# ->	true
num.instance_of? Fixnum		# ->	true
num.instance_of? Numeric	# ->	false
```

### 26.2 考察类

考察类的层次结构很容易。可以使用 `Class#superclass` 得到任何类的父类。对类和模块来说，`Module#ancestors` 会同时列出超类和混入的（mixed-in）模块。

```ruby
klass = Fixnum
begin
  print klass
  klass = klass.superclass
  print " < " if klass
end while klass
puts
p Fixnum.ancestors
```

#### 26.2.1 窥入类

可以在一个特别对象中找出关于其方法和常量的更多信息。它不是检查对象是否响应了一个给定的消息，可以通过访问级别（access level）来查找方法，可以只查找单例（singleton）方法，也可以查看对象的常量、局部变量和实例变量。

`Module.constants` 返回模块中所有可用的常量，它包括模块超类中的常量。

在 Ruby 1.8 中，默认情况下反射（reflection）方法会递归（recurse）进入父类中，同样它们的父类会沿着祖先链递归进入它们的父类中。传递 false 参数会阻止这种行为。

### 26.3 动态地调用方法

根据命令调用函数，在 Ruby 中，可以只用一行代码就完成所有事情。

```ruby
Commands.send(command_string)
```

把所有命令函数塞到一个类中，然后创建类的一个实例（commands）。它是动态的。这个 Ruby 版本可以很容易地找到在运行时添加的新方法。

无需编写特别的 command 类来支持 send 方法：它适用于任何对象。

另一种动态调用方法的方式是使用 Method 对象。

```ruby
trane = "John".method(:length)
trane.call
```

你可以像对待其他对象一样到处传递 Method 对象，当调用 `Method#call` 时，那个方法就就会运行。

你也可以和迭代器一起使用 Method 对象：

```ruby
def double(a)
  2 * a
end
mObj = method(:double)
[1, 3, 5, 7].collect(&mObj)	# ->	[2, 6, 10, 14]
```

Method 对象绑定到一个特别的对象。你可以创建非绑定方法（它的类是 UnboundMethod），然后将它们绑定到一个或多个对象。

```ruby
unbound_length = String.instance_method(:length)
class String
  def length
    99
  end
end
str = "cat"
str.length	# -> 99
bound_length = unbound_length.bind(str)
bound_length.call	# -> 3
```

还有另外一种方法可以动态地调用方法。`eval` 方法以及它的变种（`class_eval`、`module_eval` 和 `instance_eval`）会解析和执行合法 Ruby 源码的任意字符串。

```ruby
trane = %q("John Coltrane".length)
miles = %q("Miles".sub(/iles/, '.'))
eval trane	# -> 13
eval miles
```

使用 `eval` 时，明确指出表达式求值时使用的上下文而非当前上下文，是由帮助的。可以在想要的位置调用 `Kernel#binding` 来得到上下文。

#### 26.3.1 性能考虑

`eval` 会比别的技术慢很多。

### 26.4 系统钩子

钩子是一个跟踪（trap）一些 Ruby 事件的技术，比如跟踪对象创建。在 Ruby  中，最简单的 hook 技术是截取程序执行的系统命令。

```ruby
module Kernel
  alias_method :old_system, :system
  def system(*args)
    result = old_system(*args)
    puts "system(#{args.join(', ')}) returned #{result}"
	result
  end
end
```

一个更强大的钩子是捕获对象的创建。如果当创建每个对象时你都能在现场，就可以完成各种有趣的事情：你可以包装（wrap）它们，向其中添加方法，从其中删除方法，以及把它们添加到容器中以实现持久性等。

```ruby
class Object
  attr_accessor :timestamp
end
class Class
  alias_method :old_new, :new
  def new(*args)
    result = old_new(*args)
    result.timestamp = Time.now
    result
  end
end
```

方法重命名技术可能会导致无限循环，如果一个子类作同样的事情，用同样的名字对方法重命名，那么它最终会进入一个无限循环。你可以通过将方法的别名指定为一个唯一的符号名，或采用一致的命名规则来避免这个问题。

#### 26.4.1 运行时回调函数

无论何时发生了以下事件，你就会被通知。

| 事件                 | 回调函数                                |
| ------------------ | ----------------------------------- |
| 添加实例方法             | `Module#method_added`               |
| 删除实例方法             | `Module#method_removed`             |
| 取消实例方法的定义          | `Module#method_undefined`           |
| 添加 singleton 方法    | `Kernel.singleton_method_added`     |
| 删除 singleton 方法    | `Kernel.singleton_method_removed`   |
| 取消 singleton 方法的定义 | `Kernel.singleton_method_undefined` |
| 子类化某个类             | `Class#inherited`                   |
| Mixin 一个模块         | `Module#extend_object`              |

默认情况下，这些方法什么也不做。如果在类中回调函数被定义了，则它会自动地被调用。

### 26.5 跟踪程序的执行

`set_trace_func` 可以执行一个 Proc，并提供各种丰富的调试信息。

```ruby
set_trace_func proc {|event, file, line, id, binding, classname |
  printf "%8s %s:%-2d %10s %8s\n", event, file, line, id, classname
}
t = Test.new
t.test
```

#### 26.5.1 如何到这儿的

在 Ruby 中至少可以使用 caller 方法正确地找到调用栈

```ruby
caller.join("\n")
```

#### 26.5.2 源代码

`__FILE__` 这个特殊变量包含当前源文件的名称。

`Kernel.caller` 方法返回调用栈——当调用该方法时存在的栈帧（frame）的列表。表中的每一项以文件名、冒号以及文件中的行号开始。

如果程序用散列表初始化了 `SCRIPT_LINES__` 常量，散列表会得到每个文件的源代码。这些文件是通过使用 `require` 或 `load` 依次被载入到解释器中的。

### 26.6 列集合分布式 Ruby

Java 提供了序列化（serialize）对象的特性，可以在某处保存对象，并且需要的话可以重建（reconstitute）它们。

Ruby 把这种类型的序列化称为列集（marshaling）。可以使用 `Marshal.dump` 方法来保存一个对象和它的部分或所有组成对象（component）。通常你从某些给定的对象作起点，对其整个对象树进行转储，然后你可以使用 `Marshal.load` 方法来重建对象。

```ruby
c = Chord.new()
File.open("posterity", "w+") do |f|
  Marshal.dump(c, f)
end
File.open("posterity") do |f|
  chord = Marshal.load(f)
end
```

#### 26.6.1 定制序列化策略

并不是所有的对象都可以转储：绑定（bingding）、过程（procedure）对象、IO 类的实例以及 singleton 对象，不能再正运行的 Ruby 环境之外保存。

Marshal 提供了你所需要的钩子（hook）。只要在需要定制序列化的对象中实现两个实例方法：一个是 `marshal_dump`，它把对象写到字符串，另一个是 `marshal_load`，它读取字符串并用它来初始化新分配的对象。

#### 26.6.2 用 YAML 来列集

Marshal 模块內建在解释器中，它使用了二进制格式在外部保存对象。这个二进制有一个很大的不足：如果解释器有了非常大的改变，这个 marshal 二进制格式可能也会改变，老的转储文件可能再也不能被载入。

另外一个办法是使用不那么严谨（less fussy）的外部个事。其中一个选择是 YAML。

```ruby
require "yaml"
class Special
  def initialize(valuable, volatile, precious)
    @valuable = valuable
    @volatile = volatile
    @precious = precious
  end
  def to_yaml_properties
    %w{ @precious @valuable }
  end
  def to_s
    "#@valuable #@volatile #@precious"
  end
end
obj = Special.new("a","b","c")
puts "#{obj}"
data = YAML.dump(obj)
obj = YAML.load(data)
```

#### 26.6.3 分布式 Ruby

既然可以把对象或一组对象序列化到一种适合进程外（out-of-process）存储的形式，也可以使用这个能力把对象从一个进程发送到另一个进程。

建议使用分布式 Ruby 库（drb），它现在是一个标注你的 Ruby 库。

使用 drb 的 Ruby 进程可能会作为服务器、客户机或者两者都是。drb 服务器是对象的源，而客户机是对象的使用者。对客户机来说，这些对象表现为本地对象，但是实际上代码仍然在远程执行。

服务器通过将对象与一个给定的端口相关联来启动服务。线程在内部被创建来处理对这个端口的请求，因此当退出程序时，请记住等待（join）drb 线程终止。

```ruby
require "drb"
class TestServer
  def add(*args)
    args.inject {|n, v| n + v }
  end
end
server = TestServer.new
DRb.start_service("druby://localhost:9000", server)
DRb.thread.join
```

一个简单的 drb 客户端只是创建一个本地的 drb 对象，并把它与在远程服务器上的对象关联起来，这个本地对象只是一个代理。

```ruby
require "drb"
DRb.start_service()
obj = DRbObject.new(nil, "druby://localhost:9000")
puts "Sum is: #{obj.add(1, 2, 3)}"
```

客户机连接上服务器并调用 add 方法，这个方法使用 `inject` 魔法（magic）对它的参数求和。

DRbObject 的初始参数 nil 参数表明我们希望附着（attach）到一个新的分布式对象。也可以使用现有的对象。

Ho hum 是一个全功能的分布式对象机制——但是它仅用几百行 Ruby 代码写成。它没有命名（naming）服务或交易（trader）服务，或任何你能够在 CORBA 中看到的东西，但是它简单，运行起来相当快。

JavaSpaces 是基于一种被称为 Linda 的技术。Ruby 版本的 Linda 被称为 Rinda。

如果你喜欢臃肿、麻木但支持互操作的远程信息调用，可以看看 Ruby 分发中 SOAP 库。

### 26.7 编译时？运行时？任何时

Ruby 在 ”编译时” 和 “运行时” 之间没有太大的差别。它们都是相同的。可以向一个正运行的进程添加代码。可以动态地重新定义方法，把它们的作用范围从 public 改变成 private 等等。甚至可以修改像 Class 和 Object 那样的基本类型。






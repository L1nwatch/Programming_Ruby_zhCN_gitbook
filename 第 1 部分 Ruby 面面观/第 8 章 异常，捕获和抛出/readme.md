传统的做法是使用返回码。方法在失败时会返回一些特定值。然后这个值会沿着调用例程的层次往回传播，直到有函数想要处理它。问题是，管理所有这些错误码是一件痛苦的事情。

异常在很大程度上解决了这个问题。异常允许把错误信息打包到一个对象中，然后该异常对象被自动传播回调用栈（calling stack），直到运行系统找到明确声明直到如何处理这类异常的代码为止。

### 8.1 异常类

含有异常信息的数据包（package）是 Exception 类、或其子类的一个对象。Ruby 预定义了一个简洁的异常层次结构。

当需要引发（raise）异常时，可以使用某个內建的 Exception 类，或者创建自己的异常类。如果创建自己的异常类，可能你希望它从 StandardError 类或其子类派生。否则，你的异常在默认情况下不会被捕获。

每个 Exception 都关联有一个消息字符串和栈回溯信息（backtrace）。如果定义自己的异常，可以添加额外的信息。

### 8.2 处理异常

```ruby
begin
  # ...
rescue SystemCallError
  # ...
  raise
end
```

当异常被引发时，Ruby 将相关 Exception 对象的引用放在全局变量 `$!` 中，这与任何随后的异常处理不相干。

可以不带任何参数来调用 `raise`，它会重新引发 `$!` 中的异常。这是一个有用的技术，它允许我们先编写代码过滤掉一些异常，再把不能处理的异常传递到更高的层次。

在 `begin` 块中可以有多个 `rescue` 子句，可以提供一个局部变量名来接收匹配的异常。

```ruby
begin
  # ...
rescue SyntaxError, NameError => boom
  print boom
rescue StandardError => bang
  print bang
end
```

匹配使用 `parameter === $!` 完成的。对于大多数异常来说，如果 `rescue` 子句给出的类型与当前引发的类型相同，或者它是引发异常的超类（superclass），这意味着匹配是成功的。如果编写一个不带参数表的 rescue 子句，它的默认参数是 StandardError。

尽管 `rescue` 子句的参数通常是 `Exception` 类的名称，实际上它们可以是任何返回 `Exception` 类的表达式（包括方法调用）。

#### 8.2.1 系统错误

当对操作系统的调用返回错误码时，会引发系统错误。在 POSIX 系统上，这些错误名称有诸如 EAGAIN 和 EPERM 等（在 Unix 机器上，键入 `man error`，你会得到这些错误的列表）。

Ruby 得到这些错误，把每个错误包装（warp）到特定的对象中。每个错误都是 SystemCallError 的子类，定义在 Errno 模块中。这意味着，可以发现类名如：`Errno::EAGAIN`、`Errno::EIO` 和 `Errno::EPEAM` 等的异常。如果想得到低层的系统错误码，则每个 Errno 异常对象有一个 Errno 的类常量（class constant），它包含相应的系统错误码。

有时候会出现两个常量映射到相同的错误码。如果你要求 rescue 其中一个错误，那么另一个也会被 rescue。通过重定义 `SystemCallError#===` 可以做到这一点。

#### 8.2.2 善后

`ensure` 跟在最后的 `rescue` 子句后面，它包含一段当 block 退出时总是要被执行的代码。不管 block 是否正常退出，是否引发并 rescue 异常，或者是否被未捕获的异常终止——这个 ensure 总会得到运行。

`else` 会出现在 `rescue` 子句之后和任何一个 `ensure` 子句之前，只有当主代码体没有引发任何异常时才会被执行。

#### 8.2.3 再次执行

可以在 `rescue` 子句中使用 `retry` 语句去重复执行整个 `begin/end` 区域。显然这很可能会导致无限循环。

### 8.3 引发异常

可以使用 `Kernel.raise` 方法在代码中引发异常（或者 `Kernel.fail`）

```ruby
raise
raise "bad mp3 encoding"
raise InterfaceException, "Keyboard failure", caller
```

第一种形式只是简单地重新引发当前异常（如果没有当前异常的话，引发 RuntimeError）。这种形式用于首先截获异常再将其继续传递的异常处理方法中。

第二种形式创建新的 RuntimeError 异常，把它的消息设置为指定的字符串。然后异常随着调用栈向上引发。

第三种形式使用第一个参数创建异常，然后把相关联的消息设置给第二个参数，同时把栈信息（trace）设置给第三个参数。通常，第一个参数是 Exception 层次结构中某个类的名称，或者是某个异常类的对象实例的引用。通常使用 `Kernel.caller` 方法产生栈信息。

下面的代码通过只将调用栈的子集传递给新异常，从而达到从栈回溯信息中删除两个函数的目的。

```ruby
raise ArgumentError, "Name too big", caller[1..-1]
```

#### 8.3.1 添加信息到异常

你可以定义自己的异常，保存任何需要从错误发生地传递出去的信息。

```ruby
class RetryException < RuntimeError
  attr :ok_to_retry
  def initialize(ok_to_retry)
    @ok_to_retry = ok_to_retry
  end
end
```

### 8.4 捕获和抛出

想要在正常处理过程期间能够从一些深度嵌套的结构中跳转出来，`catch` 和 `throw` 可以做到这点。

```ruby
catch (:done) do
  while line = gets
    throw :done unless fields = line.split(/\t/)
    songlist.add(Song.new(*fields))
  end
  songlist.play
end
```

`catch` 定义了以给定名称（可能是符号或字符串）为标签的 block。这个 block 会正常执行直到遇到 `throw` 为止。

当 Ruby 碰到 `throw`，它迅速回溯（zip back）调用栈，用匹配的符号寻找 `catch` 代码块。当发现它之后，Ruby 将栈清退（unwind）到这个位置并终止该 block。如果调用 `throw` 时指定了可选的第二个参数，这个值会作为 `catch` 的值返回。

```ruby
def prompt_and_get(prompt)
  print(prompt)
  res = readline.chomp
  throw :quit_requested if res == "!"
  res
end

catch :quit_requested do
  name = prompt_and_get("Name: ")
  age = prompt_and_get("Age: ")
  sex = prompt_and_get("Sex: ")
  # ...
end
```

`throw` 没必要出现在 `catch` 的静态作用域中。
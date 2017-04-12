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


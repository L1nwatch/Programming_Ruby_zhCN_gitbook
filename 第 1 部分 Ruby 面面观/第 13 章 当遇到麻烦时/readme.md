### 13.1 Ruby 调试器

Ruby 自带一个调试器，并被集成到 Ruby 的基本系统中。

```ruby
Ruby -r debug [调试选项] [程序文件] [程序参数]
```

调试器支持你所期望的常用功能，包括设置断点、步入或者跳过方法调用以及显示调用栈和变量。它还能列出某个对象或类中定义的实例方法，也能列出并控制 Ruby 中单独的线程。

如果在安装 Ruby 时启用了 `readline` 功能，就可以用方向键在命令的历史记录中来回选择。

```shell
ruby -r debug t.rb
```

一些基本命令：

```shell
list 1-9
b 2
c
dlsp n
del 1
watch n == 1
where
```

### 13.2 交互式 Ruby

```shell
irb [irb 选项] [ruby 脚本] [程序参数]
```

`irb` 中可以创建子会话，每个会话还可以有自己的上下文。

### 13.3 编辑器支持

Ruby 解释器被设计成一次读一个程序，这意味着你可以通过管道把整个程序作为标准输入传递给解释器，而它就可以很好地工作。

在 `vi` 编辑器中你也可以做类似的事情：`%!ruby` 会用代码运行的结果替换程序文本，`:w !ruby` 会显示输出而不影响缓冲区。

### 13.4 但是它不运作

常见问题和技巧：

*   运行你的脚本时启用警告（`-w` 命令行选项）
*   Block 的参数和局部变量处于相同的作用域中。当执行 block 时，如果 block 的参数存在已定义的同名局部变量，那么该变量可能被 block 修改
*   终端输出可能被缓存。另外，如果同时向 `$stdout` 和 `$stderr` 输出信息，那么输出的顺序与你所期望的可能有所不同。始终用非缓冲的 I/O 来处理调试用信息（设置 `sync=true`）
*   数字可能是字符串，可以使用 `map` 来一次性转换所有的字符串：

```ruby
while line = gets
  num1, num2 = line.split(/,/).map {|val| Integer(val) }
  # ...
end
```

*   无意的别名——如果你使用一个对象作为散列表的关键字，确保它不会改变其 hash 值（或者改变后调用 `Hash#rehash`）

```ruby
arr = [1, 2]
hash = { arr => "value"}
hash[arr]	# ->	"value"
arr[0] = 99
hash[arr]	# ->	nil
hash.rehash	# ->	{[99, 2] => "value"}
hash[arr]	# ->	"value"
```

*   使用 `Object#freeze`，如果你怀疑某段代码改变了变量，可以试着冻结该变量来进行定位

### 13.5 然而它太慢了

`Benchmark` 模块和 `Ruby` 的剖析器（profiler）可以帮助我们。

### 13.5.1 Benchmark

可以使用 `Benchmark` 模块来对代码段计时。

使用 `benchmark` 时需要很小心，因为垃圾回收常常会导致 Ruby 程序运行速度变慢。垃圾回收可能在程序运行的任何时候发生，因此 `benchmark` 可能会给你错误的结果：`benchmark` 显示一段代码运行很慢，而实际上速度变慢是由于执行该代码时垃圾回收恰好被触发所造成的。`Benchmark` 模块的 `bmbm` 方法会运行测试两次，一次作为预演，一次实际测量性能，力图将垃圾回收带来的影响降到最低。`Bencemark` 测试过程本身相对设计得比较优雅——它不会使程序运行慢太多。

```ruby
require "benchmark"
include Benchmark
LOOP_COUNT = 1_000_000

bm(12) do |test|
  test.report("normal:") do
    LOOP_COUNT.times do |x|
      y = x + 1
    end
  end
  test.report("predefine:") do
    x = y = 0
    LOOP_COUNT.times do |x|
      y = x + 1
    end
  end
end
```

#### 13.5.2 剖析器

Ruby 自带一个代码剖析器，可以告诉你程序中的每个方法调用的次数以及 Ruby 运行这些方法的平均时间和累计时间。

可以使用 `-r profile` 命令行选项，也可以在代码中使用 `require 'profile'` 将 `profile` 添加到代码中。

注意耗时会比不使用剖析器运行程序要慢得多。剖析对性能影响很大，但是假设该影响是全局的，因此获得的相对的值仍然是有意义的。

注意，有时候剖析器引起的速度降低可能会掩盖其他问题。
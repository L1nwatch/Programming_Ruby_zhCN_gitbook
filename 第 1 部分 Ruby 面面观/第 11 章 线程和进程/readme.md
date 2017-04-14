### 11.1 多线程

Ruby 线程是彻底可移植的——它们不依赖于操作系统。当然同时也无法获得本地线程（native thread）所带来的某些益处。

Ruby 线程运行在一个进程内、并且是单独的一个本地线程内，它们被约束每次只能运行在一个处理器上。

#### 11.1.1 创建 Ruby 线程

```ruby
require "net/http"
pages = %w(www.rubycentral.com slashdot.org www.google.com)
threads = []
for page_to_fetch in pages
  # 注意这里不要直接用 page_to_fetch, 会被其他线程改变
  threads << Thread.new(page_to_fetch) do |url|	
    h = Net::HTTP.new(url, 80)
    puts "Fetching: #{url}"
    resp = h.get('/', nil)
    puts "Got #{url}: #{resp.message}"
  end
end
threads.each {|thr| thr.join}
```

线程共享在它开始执行时就已经存在的所有全局变量、实例变量和全局变量中。

在线程 block 里创建的局部变量对线程来说是真正的局部变量——每个线程会有这些变量的私有备份。

##### 操作线程

当 Ruby 程序终止时，不管线程的状态如何，所有的线程都被杀死。可以通过调用线程的 `Thread#join` 方法来等待特定线程的正常结束。调用 `join` 的线程会阻塞，直到指定的线程正常结束为止。如果不想永远阻塞，可以给 `join` 一个超时参数。`join` 的另一个变种，`Thread#value` 方法返回线程执行的最后语句的值。

使用 `Thread.current` 总是可以得到当前线程。可以使用 `Thread.list` 得到一个所有线程的列表，返回包含所有可运行或被停止的线程对象。可以使用 `Thread#status` 和 `Thread#alive?` 去确定特定线程的状态。

另外可以使用 `Thread#priority=` 调整线程的优先级。更高优先级的线程会在低优先级的线程前面运行。

##### 线程变量

如果需要线程局部变量能被别的线程访问（包括主线程），Thread 类提供了一种特别的措施，允许通过名字来创建和访问线程局部变量。可以简单地把线程对象看做一个散列表，使用 `[]=` 写入元素并使用 `[]` 把它们读出。

```ruby
count = 0
threads = []
10.times do |i|
  threads[i] = Thread.new do
    sleep(rand(0.1))
    Thread.current["mycount"] = count
    count += 1
  end
end
threads.each {|t| t.join; print t["mycount"], ", "}
puts "count = #{count}"
```

#### 11.1.2 线程和异常

如果线程引发（raise）了未处理的异常，接下来的操作依赖于 `abort_on_exception` 标志和解释器 `debug` 标志的设置。

如果 `abort_on_exception` 是 `false`，`debug` 标志没有启用（默认条件），未处理的异常会简单地杀死当前线程——而所有其他线程继续运行。实际上，除非对引发这个异常的线程调用了 `join`，你甚至根本不知道这个异常存在。

当线程被 `join` 时，我们可以 `rescue` 这个异常。

设置 `abort_on_exception` 为 true（`Thread.abort_on_exception = true`），或者使用 `-d` 选项去打开 `debug` 标志，未处理的异常会杀死所有正在运行的线程。

在循环里面，线程使用 `print` 而不是 `puts` 去输出。原因是 `puts` 的工作被分为两部分：输出其参数，然后再输出回车换行符。这两个动作之间可能切换线程导致输出交织在一起。

### 11.2 控制线程调度器

多线程程序中建立时间依赖性（`timing dependency`）通常被认为是糟糕的设计，这会导致代码复杂化同时阻碍线程调度器优化程序的执行。

有时候需要显式地控制线程，Thread 类提供了若干种控制线程调度器的方法。调用 `Thread.stop` 停止当前线程，调用 `Thread#run` 安排运行特定的线程，`Thread.pass` 把当前线程调度出去，允许运行别的线程，`Thread#join` 和 `Thread#value` 挂起调用它们的线程，直到指定的线程结束为止。

但是在实际的代码中使用这些原语（primitive）实现同步并不是一件容易的事情。

### 11.3 互斥

这是最底层的阻止其他线程运行的方法，它使用了全局的线程关键（thread-critical）条件。当条件被设置为 `true`（使用 `Thread.critical=` 方法）时，调度器将不会调度现有的线程去运行。但是这不会阻止创建和运行新线程。某些线程操作（如停止或杀死线程，在当前线程中睡眠和引发异常）会导致即使线程处于一个关键区域内也会被调度出去。

直接使用 `Thread.critical=` 当然是可以的，但是它不是很方便。Ruby 有多种变通方法，其中的一种是 `Monitor` 库，还有 `Sync` 库、`Mutex_m` 库、`thread` 库中的 `Queue` 类。

#### 11.3.1 监视器


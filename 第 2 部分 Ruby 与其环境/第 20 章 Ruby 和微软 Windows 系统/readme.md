Ruby 来自于以 Unix 为中心的人，但是这些年它也已经在 Windows 世界开发了许多有用的特性。

### 20.1 得到 Ruby for Windows

两种版本的 Ruby 在 Windows 环境下可用。

第一种是运行在本地的 Ruby 版本，即它只是另外一个 Windows 程序。下载这个版本最简单的方式是运行 `One-Click Installer`。

另一种方法可以从源文件编译 Ruby。

第二种版本使用称为 Cygwin 的仿真层，它在 Windows 系统上提供了类似 Unix 的环境。Cygwin 版本的 Ruby 与运行在 Unix 平台上的 Ruby 最接近，但是运行它意味着也必须要安装 Cygwin。

推荐 Ruby 的本地（native）版本。

### 20.2 在 Windows 下运行 Ruby

`ruby.exe` 用于命令行提示（DOS shell），这和 Unix 版本中的 Ruby 可执行程序是一样的。对于读写标准输入和输出的程序来说，是挺不错的。但这意味着，任何时候运行 `ruby.exe`，你都会得到一个 DOS shell，即使你不需要它。

使用 `rubyw.exe`，它与 `ruby.exe` 一样，除了不会提供标准输入、标准输入或标准错误外，并且当运行时也不会启动 DOS shell。

### 20.3 Win32API

如果你需要直接使用一些 Windows 32 API 函数，或者需要使用其他 DLL 中的某些入口函数（entry point），可以使用 Win32API 库。

```ruby
arg = "ids=#{resp.int1_orders.join(",")}"
fname = "/temp/invoices.pdf"
site = Net::HTTP.new(HOST, PORT)
site.use_ssl = true
http_resp, = site.get2("/fulfill/receipt.cgi?" + arg,
  "Authorization" => "Basic " +
    ["name:passwd"].pack("m").strip )
File.open(fname, "wb") { |f| f.puts(http_resp.body) }
shell = Win32API.new("shell32", "ShellExecute",
  ["L","P","P","P","P","L"], "L")
shell.Call(0, "print", fname, 0, 0, SW_SHOWNORMAL)
```

这里创建了一个 Win32API 对象，它表示对特定 DLL 入口函数的调用，这是由函数名称、含有该函数的 DLL 名称以及函数原型特征（参数类型和返回类型）来指定的。

DLL 函数的许多参数是某种形式的二进制结构。Win32API 通过使用 Ruby 的 String 对象来回传递这些二进制数据。必要的话你需要对这些字符串打包和拆包。

### 20.4 Windows 自动化

一个称为 WIN32OLE 的 Ruby 扩展，使得可以把 Ruby 当做 Windows 自动化的一个客户程序来使用。Win32OLE 是标准 Ruby 发行版的一部分。

Windows 自动化允许自动化控制器（automation controller，亦即客户端程序）对自动化服务器发出命令和查询，自动化服务器可能是微软 Excel、Word、PowerPoint 等等。

若要执行自动化服务器中的方法，可以通过从 WIN32OLE 对象中调用相同名称的方法来实现。

比如，可以创建一个新的 WIN32OLE 客户程序，运行全新的 Internet Explorer（IE）拷贝：

```ruby
ie = WIN32OLE.new("InternetExplorer.Application")
ie.visible = true
ie.gohome
ie.navigate("http://www.baidu.com")
```

那些 WIN32OLE 不知道的方法（比如 visible、gohome 或 navigate）会被传递到 `WIN32OLD#invoke` 方法，会将正确的命令发送给服务器。

#### 20.4.1 得到的设置属性

使用普通的 Ruby 散列表表示法可以从服务器中设置和得到属性。

```ruby
excel = WIN32OLE.new("excel.application")
excelchart = excel.Charts.Add()
# ...
excelchart["Rotation"] = 45
puts excelchart["Rotation"]
```

OLD 对象的参数作为 WIN32OLE 对象的属性会被自动设置。这意味着可以通过对一个对象属性进行赋值来设置参数。

下面的代码启动 Excel，创建一个图标，然后在屏幕上旋转它：

```ruby
require "win32ole"
ChartTypeVal = -4100
excel = WIN32OLE.new("excel.application")
excel["Visible"] = TRUE
excel.Workbooks.add()
excel.Range("a1")["Value"] = 3
excel.Range("a2")["Value"] = 2
excel.Range("a3")["Value"] = 1
excel.Range("a1:a3").Select()
excelchart = excel.Charts.Add()
excelchart["Type"] = ChartTypeVal
30.step(180, 5) do |rot|
  excelchart.rotation = rot
  sleep(0.1)
end
excel.ActiveWorkbook.Close(0)
excel.Quit()
```

#### 20.4.2 命名参数

其他自动化客户语言如 Visual Basic 有命名参数这个概念。

在 Ruby 中，可以通过传递带有命名参数的散列表来使用这个特性。

```ruby
Song.new("title" => "Get It On")
```

#### 20.4.3 for each

Visual Basic 有 "for each" 语句，它对服务器中项（item）的收集（collection）进行迭代，WIN32OLE 对象有 each 方法（它接受一个代码块）去完成同样的事情。

```ruby
require "win32ole"
excel = WIN32OLE.new("excel.application")
excel.Workbooks.Add
excel.Range("a1").Value = 10
excel.Range("a2").Value = 20
excel.Range("a3").Value = "=a1+a2"
excel.Range("a1:a3").each do |cell|
  p cell.Value
end
```

#### 20.4.4 事件

以 Ruby 编写的自动化客户程序可以注册自己去接收来自其他程序的事件。

#### 20.4.5 优化

使用 WIN32OLE，需要当心那些不必要的动态查找（lookup）。只要有可能，最好把 WIN32OLE 赋值给变量，然后使用它来引用其元素，而不是：

```ruby
workbook.Worksheets(1).Range("A1").value = 1
workbook.Worksheets(1).Range("A2").value = 2
workbook.Worksheets(1).Range("A3").value = 4
workbook.Worksheets(1).Range("A4").value = 8
```

可以通过把表达式的前面部分保存到临时变量中，以去掉这些共同的子表达式，再从这个变量那儿进行调用。

```ruby
worksheet = workbook.Worksheets(1)
worksheet.Range("A1").value = 1
worksheet.Range("A2").value = 2
worksheet.Range("A3").value = 4
worksheet.Range("A4").value = 8
```

也可以为特定的 Windows 类型库创建 Ruby 存根（stub）。这些存根把 OLE 对象包装到 Ruby 类中，其中 OLE 对象中的每个入口函数都在该类中有一种对应的方法。在内部，存根方法使用入口函数的编号而不是它的名称，这加快了访问速度。

类型库的外部方法和事件会作为 Ruby 方法写入到指定的文件中。你可以把它包含在你的程序中，并直接调用这些方法。

#### 20.4.6 更多的帮助

如果需要 Ruby 访问 Windows NT、2000 或 XP 的话，你可以看一下 Daniel Berger 的 Win32Utils 项目。在那里会找到访问 Windows 剪切板、事件日志和调度器等的模块。

另外，DL 库允许 Ruby 程序调用动态装入的共享库中的方法。在 Windows 上，这意味着 Ruby 代码可以装入和调用 Windows DLL 中的入口函数。

```ruby
require "dl"
User32 = DL.dlopen("user32")
MB_OKCANCEL = 1
message_box = User32["MessageBoxA", "ILSSI"]
r, rs = message_box.call(0, "OK?", "Please Confirm", MB_OKCANCEL)
case r
  when 1
  print("OK!\n")
  when 2
  print("Cancel!\n")
end
```

打开 User32 DLL 这段代码。创建 `message_box` 这个 Ruby 对象，该对象包装（wrap）了 MessageBoxA 的1入口函数。第二个参数 “ILSSI” 声明这个方法会返回一个整数，同时接受一个 Long、两个字符串和一个整数作为它的参数。

这个包装（wrapper）对象被用来调用在 DLL 中的消息框入口函数。返回值是用户交互的结果，以及传入参数的一个数组。


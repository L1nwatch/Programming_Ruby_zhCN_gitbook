Ruby 应用程序归档（Application Archive）含有几个提供图形用户界面（GUI）的扩展，包括对 Fox、GTK 以及其他一些部件库（widget）的支持。

Tk 扩展被集成到了 Ruby 主发行版中，即可以运行在 Unix 上，也可以运行在 Windows 系统上。

TK 以一种组合模型（composition model）工作——你开始先创建一个容器（TKFrame 或 TKBoot），然后在其中创建部件（widget、GUI 组件的另一种说法），例如按钮或者标签（label）。当你准备好启动 GUI 时，调用 `Tk.mainloop`。最后 Tk 引擎会获得程序的控制权，显示各个部件并依据 GUI 事件调用相应的代码。

### 19.1 简单的 Tk 应用程序

```ruby
require "tk"
root = TkRoot.new { title "Ex1" }
TKLabel.new(root) do
  text "Hello, World!"
  pack("padx" => 15, "pady" => 15, "side" => "left")
end
Tk.mainloop
```

明确指定根窗框是个好习惯，但是你也可以省略它——以及额外的选项：

```ruby
require "tk"
TKLabel.new { text "Hello, World!"; pack }
Tk.mainloop
```

### 19.2 部件

使用 new 方法可以创建部件实例。如果不为给定的部件指定一个父部件，那么父部件默认为根窗框。

#### 19.2.1 设置部件选项

部件的选项列表通常带一个连字符。在 Perl/Tk 中，选项被传递给部件的一个 Hash 中。在 Ruby 中你也可以这样做，但也可以使用代码 block 来传递选项：在 block 中选项的名字被用作方法名，选项的参数作为方法的参数。部件以它的父部件作为第一个参数，后跟可选的选项 Hash 或者可选的选项 block。

```ruby
TkLabel.new(parent_widget) do
  text "Hello, World!"
  pack("padx" => 5,
    "pady" => 5,
    "side" => "left")
end
# or
TkLabel.new(parent_widget, "text" => "Hello, World!").pack(...)
```

使用代码 block 形式时需要注意，变量的作用域并不是你所想的那样。Block 实际不在调用者上下文而是在部件对象上下文中被处理的。这意味着不能在 block 中访问调用者的实例变量，但是可以访问外围作用域中的局部变量和全局变量。

#### 19.2.2 获取部件数据

通过使用回调函数和绑定变量，可以从部件中获取信息。

command 选项以 Proc 对象为参数，当回调函数被触发时，该对象将被调用：

```ruby
require "tk"
TkButton.new do
  text "EXIT"
  command {exit}
  pack("side" => "left", "padx" => 10, "pady" => 10)
end
Tk.mainloop
```

还可以使用 TkVariable 代理，将某个 Ruby 变量绑定到 Tk 部件的值。这样的安排使得每当部件的值发生变化时，Ruby 变量也会自动被更新，而当变量发生变化时，部件也会改变以反映新的值。

```ruby
require "tk"
packing = {"padx" => 5, "pady" => 5, "side" => "left"}
checked = TkVariable.new
def checked.status
  value == "1" ? "Yes" : "No"
end
status = TkLabel.new do
  text checked.status
  pack(packing)
end
TkCheckButton.new do
  variable checked
  pack(packing)
end
TkButton.new do
  text "Show status"
  command { status.text(checked.status)}
  pack(packing)
end
Tk.mainloop
```

#### 19.2.3 动态地设置/获取选项

除了在创建部件时可以设置选项外，还可以在运行时重新配置部件。每个部件都支持 `configure` 方法，它和 `new` 一样以一个 Hash 对象或者代码 block 作为参数。

```ruby
require "tk"
root = TkRoot.new { title "Ex3" }
top = TkFrame.new(root) { relief "raised"; border 5 }
lbl = TkLabel.new(top) do
  justify "center"
  text "Hello, World!"
  pack("padx" => 5, "pady" => 5, "side" => "top")
end
TkButton.new(top) do
  text "Ok"
  command {exit}
  pack("side" => "left", "padx" => 10, "pady" => 10)
end
TkButton.new(top) do
  text "Cancel"
  command { lb.configure("text" => "Goodbye, Cruel World!")}
  pack("side" => "right", "padx" => 10, "pady" => 10)
end
top.pack("fill" => "both", "side" => "top")
Tk.mainloop
```

可以使用 `cget` 来查询部件的具体选项：

```ruby
require "tk"
b = TkButton.new do
  text "OK"
  justify "left"
  border 5
end
b.cget("text")		# =>	"OK"
b.cget("justify")	# =>	"left"
b.cget("border")	# =>	5
```

#### 19.2.4 示例应用程序

>   布局管理
>
>   部件方法 pack 告诉布局管理器会根据我们指定的条件来布局部件。布局管理器能识别 3 个命令：
>
>   | 命令    | 位置说明        |
>   | ----- | ----------- |
>   | pack  | 灵活的、基于限制的位置 |
>   | place | 绝对位置        |
>   | grid  | 表格（行/列）位置   |

### 19.3 绑定事件

你可以使用部件的 bind 方法创建一个从特定部件事件到代码 block 的绑定

```ruby
require "tk"
image1 = TkPhotoImage.new {file "img1.gif"}
image2 = TkPhotoImage.new {file "img2.gif"}
b = TkButton.new(@root) do
  image image1
  command {exit}
  pack
end
b.bind("Enter") { b.configure("image" => image2)}
b.bind("Leave") { b.configure("image" => image1)}
Tk.mainloop
```

在 bind 函数的参数中，事件名字可以被短线分割为几个子串的组成部分，顺序为 **修饰符 - 修饰符 - 类型 - 细节**（modifier-modifier-type-detail）。Tk 参考中列出了修饰符，包括 Button1、Control、Alt、Shift 等等。类型是事件的名字（使用 X11 中的命名传统），包括 ButtonPress、KeyPress 和 Expose。

当按下 control 键并释放鼠标的 button1 时，触发的绑定可以记为：

```shell
Control-Button1-ButtonRelease
# or
Control-ButtonRelease-1
```

事件本身可以含有一些域（field），如事件发生的时间、x 和 y 位置等。bind 可以使用 event field code 将这些信息传递给回调函数。使用它们的方式类似于 printf 规范。

```ruby
canvas.bind("Motion", lambda {|x, y| do_motion(x, y)}, "%x %y")
```

### 19.4 画布

Tk 提供了一个 Canvas 部件，使用它可以绘制并生成 PostScript 输出。

```ruby
require "tk"
class Draw
  def do_press(x, y)
    @start_x = x
    @start_y = y
    @current_line = TkcLine.new(@canvas, x, y, x, y)
  end
  def do_motion(x, y)
    if @current_line
      @current_line.coords @start_x, @start_y, x, y
    end
  end
  def do_release(x, y)
    if @current_line
      @current_line.coords @start_x, @start_y, x, y
      @current_line.fill "black"
      @current_line = nil
    end
  end
  def initialize(parent)
    @canvas = TkCanvas.new(parent)
    @canvas.pack
    @start_x = @start_y = 0
    @canvas.bind("1", lambda {|e| do_press(e.x, e.y)})
    @canvas.bind("B1-Motion", lambda {|x, y| do_motion(x, y)}, "%x %y")
    @canvas.bind("ButtonRelease-1", lambda {|x, y| do_release(x, y)}, "%x %y")
  end
end
root = TkRoot.new { title "Canvas" }
Draw.new(root)
Tk.mainloop
```

### 19.5 滚动

TkCanvas、TkListbox 和 TkText 都可以被设置成使用滚动条。

滚动条和部件之间的通信是双向的。移动滚动条意味着部件视图将改变；而当通过其他方式改变部件视图时，滚动条也会相应地移动以反映新的位置。

```ruby
list_w = TkListbox.new(frame) do
  selectmode "single"
  pack "side" => "left"
end
list_w.bind("ButtonRelease-1") do
  busy do
    filename = list_w.get(*list_w.curselection)
    tmp_img = TkPhotoImage.new { file filename }
    scale = tmp_img.height / 100
    scale = 1 if scale < 1
    image_w.copy(tmp_img, "subsample" => [scale, scale])
    image_w.pack
  end
end
scroll_bar = TkScrollbar.new(frame) do
  command {|*args| list_w.yview *args }
  pack "side" => "left", "fill" => "y"
end
list_w.yscrollcommand {|first, last| scroll_bar.set(first, last)}
```

#### 19.5.1 还有一件事

你曾经见过创建了“忙光标”而忘记重置它的应用程序吗？在 Ruby 中有一个优雅的技巧可以避免此类事情的发生。我们可以使用 `busy` 方法来解决。

这里有一个例子：简单的 GIF 图像查看器（代码略）

### 19.6 从 Perl/Tk 文档转译

你可以很容易地将 Perl/Tk 文档转译为 Ruby 文档，但也有几个异常：某些方法没有实现，也可能某些额外功能没有文档。

记住选项可能以散列表的方式或者代码 block 的风格出现，并且代码 block 的作用域是在使用它的 TkWidget 内而不是类的实例内。

#### 19.6.1 对象创建

在 Perl/Tk 世界中，父部件负责创建它的子部件。而在 Ruby 中，父部件被作为第一个参数传递给子部件的构造函数。

#### 19.6.2 选项

记住 block 的作用于是不同的。

#### 19.6.3 变量引用

使用 TkVariable 将一个 Ruby 变量和一个部件的属性值关联起来。然后你可以使用 TkVariable 中的值访问方法（TkVariable#value 和 TkVariable#value=）来直接更改部件的内容。


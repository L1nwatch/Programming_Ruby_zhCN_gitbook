### 15.1 命令行

irb 是从命令行运行的。

```shell
irb [irb-options] [ruby-script] [program arguments]
```

通常你在运行 irb 时并不指定任何选项，但是如果你希望运行一个脚本并实时查看每一步的记录，可以指定 Ruby 脚本的名称以及该脚本的选项。

irb 包括一个 Ruby 解析器，因此它知道语句尚未结束。这时，提示符会变成一个星号。你可以键入 exit 或 quit，或者通过输入一个文件结束符号（如果没有设置 `IGNORE_EOF` 模式）来退出 irb。

在 irb 的会话中，你所做的工作在 irb 工作区（workspace）中累积起来。你设置的变量、定义的方法和创建的类，都被记录下来并可以被后续使用。

`load` 允许我们多次加载同一个文件，这样如果我们发现了一个 bug 然后编辑文件，可以将它重新加载到 irb 会话中。

#### 15.1.1 Tab 补齐

如果你的 Ruby 安装支持 readline，可以使用 irb 的完成功能。

Tab 补齐是以一个扩展库来实现的，即 `irb/completion`。当调用 irb 时，你可以从命令行加载它。

还可以在 irb 运行时加载补齐库。

```shell
require "irb/completion"
```

最便捷的方式是将 `require` 命令放到 `.irbrc` 文件中。

#### 15.1.2 子会话

irb 支持多个、并发的会话。当前会话只有一个；其他的在被激活前处于休眠状态。在 irb 内输入 irb 命令会创建一个子会话，输入 jobs 命令列出所有的会话，输入 fg 则激活一个特定的休眠会话。

`-r` 命令行选项，会在 irb 启动之前加载指定的文件。

#### 15.1.3 子会话与绑定

如果当你创建子会话时指定了一个对象，它会称为绑定中 self 的值。

```shell
% irb
> self
# => main
> irb "wombat"
> self
# => "wombat"
```

### 15.2 配置

irb 是高度可配置的。你可以在命令行或者从初始化文件中，甚至在 irb 内时，设置配置选项。

#### 15.2.1 初始化文件

irb 使用一个初始化文件，你可以在其中设置常用的选项、或者执行任何所需的 Ruby 语句。当 irb 开始运行时，它会尝试加载一个初始化文件，按下面的顺序查找：

```shell
~/.irbrc、.irbrc、irb.rc、$irbrc
```

在初始化文件中，你可以运行任意的 Ruby 代码。还可以设置配置项的值。配置变量在初始化文件中使用的值是符号（Symbol，以一个冒号开头）。你使用这些符号来设置散列表 `IRB.conf` 中的值。

例如，让 SIMPLE 成为所有 irb 会话的默认提示符，你可以在初始化文件中设置如下：

```shell
IRB.conf[:PROMPT_MODE] = :SIMPLE
```

配置 irb 的一个有趣技巧，你可以将 `IRB.conf[:IRB_RC]` 设置为一个 Proc 对象。只要 irb 的上下文有所变动，这个 Proc 就会被调用，并且以该上下文的配置作为参数。你可以使用这项功能，基于上下文动态地改变配置。例如：

```ruby
IRB.conf[:IRB_RC] = proc do |conf|
  leader = " " * conf.irb_name.length
  conf.prompt_i = "#{conf.irb_name} --> "
  conf.prompt_s = leader + ' \-" '
  conf.prompt_c = leader + ' \-+ '
  conf.return_format = leader + " ==> %s\n\n"
  puts "Welcome!"
end
```

#### 15.2.2 扩展 irb

因为你在 irb 中键入的内容被当做 Ruby 代码来解释，你可以通过定义新的顶层（top-level）方法来扩展 irb。

如果你在 `.irbrc` 文件中添加下面的代码，将会添加一个叫做 ri 的方法，使用传入的参数调用外部的 ri 命令：

```ruby
def ri(*names)
  system(%{ri #{names.map {|name| name.to_s}.join(" ")}})
end
```

当你下一次启动 irb 时，你将能够使用这个方法来得到参考文档

```ruby
irb > ri Proc
```

#### 15.2.3 交互式配置

多数配置值是你在运行 irb 时可设置的。

```ruby
irb > conf.prompt_mode = :SIMPLE
```

#### 15.2.4 irb 的配置选项

略

### 15.3 命令

在 irb 提示符下，你可以输入任何有效的 Ruby 表达式并查看结果。你还可以使用下面的任何命令来控制 irb 会话

```shell
exit、quit、irb_quit、irb_exit
```

退出 irb 的会话或子会话。如果你使用 cb 更改了绑定，则从绑定模式退出。

```shell
conf、context、irb_context
```

显示当前的配置。通过调用 conf 的方法来修改配置。例如，修改默认提示符：

```shell
irb > conf.prompt_i = "Yes, Master? "
```

略

#### 15.3.1 配置提示符

提示符的集合保存在提示符的散列表 `IRB.conf[:PROMPT]` 中

略

### 15.4 限制

因为 irb 的工作方式，它和标准的 Ruby 解释器稍有不兼容。问题在于局部变量的判定。

通常，Ruby 查看赋值语句来判定某个名字是否为一个变量——如果一个名字没有被赋值过，那么 Ruby 假定该名字是一个方法调用。

```ruby
eval "var = 0"
var
```

这种情况下，赋值出现在一个字符串中，因此 Ruby 并不会对它有所考虑。而另一方面：

```ruby
irb > eval "var = 0"	# =>	0
irb > var				# =>	0
```

如果你需要更紧密地匹对 Ruby 的行为，你可以将这些语句放在 `begin/end` 对中。

### 15.5 rtags 与 xmp


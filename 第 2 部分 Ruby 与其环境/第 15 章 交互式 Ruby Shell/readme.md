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
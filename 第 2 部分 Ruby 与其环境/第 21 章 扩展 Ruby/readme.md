用 Ruby 编写代码来扩展 Ruby 新的特性很简单。但是如果用 C 编写的底层代码扩展 Ruby，更有无限的可能性。

用 C 扩展 Ruby 很简单，工作就是构造一个 Ruby 对象来调用适当的 C 函数。

### 21.1 你的第一个扩展

接下来要编写的扩展与下面的 Ruby 类有相同的功能：

```ruby
class MyTest
  def initialize
    @arr = Array.new
  end
  def add(obj)
    @arr.push(obj)
  end
end
```

换言之，要用 C 编写一个扩展，让它和上面的 Ruby 类做到插入兼容（plug-compatible）。等价的 C 代码：

```c
#include "ruby.h"
static int id_push;
static VALUE t_init(VALUE self) {
  VALUE arr;
  arr = rb_ary_new();
  rb_iv_set(self, "@arr", arr);
  return self;
}
static VALUE t_add(VALUE self, VALUE obj) {
  VALUE arr;
  arr = rb_iv_get(self, "@arr");
  rb_funcall(arr, id_push, 1, obj);
  return arr;
}
VALUE cTest;
void Init_my_test() {
  cTest = rb_define_class("MyTest", rb_cObject);
  rb_define_method(cTest, "initialize", t_init, 0);
  rb_define_method(cTest, "add", t_add, 1);
  id_push = rb_intern("push");
}
```

首先，我们需要包含头文件 `ruby.h` 获得所需的 Ruby 定义。

每个扩展都定义一个名为 `Init_name` 的 C 全局函数，这个函数在解释器第一次加载（或者静态链接）名为 name 的扩展时被调用。它被用来初始化扩展，并加入到 Ruby 的环境中。

将 `add` 和 `initialize` 设置为 MyTest 的两个实例方法。调用 `rb_define_method` 会在 Ruby 方法名和实现它的 C 函数间建立一个绑定。如果 Ruby 代码调用对象实例的 `add` 方法，解释器会调用 C 函数 `t_add`，并传入一个参数。

类似地，当调用该类的 new 方法时，Ruby 会构造一个基本的对象，然后调用 `initialize`，这里定义了它要调用的 C 函数 `t_init`，而没有（Ruby 类型的）参数。

除了所有的 Ruby 参数之外，每个方法会被传入一个初始的 VALUE 参数，其中包括该方法的接收者（receiver，等同于 Ruby 代码中的 self）。

如果你编写的 Ruby 代码引用一个不存在的实例变量，则会创建它。

**警告**：每个从 Ruby 调用的 C 函数必须返回一个 VALUE，即使它只是 Qnil。否则，结果很可能是 core dump（或 GPF，General Protection Fault，一般性保护错误）。

最后，`t_add` 方法从当前对象得到实例变量 `@arr`，并调用 `Array#push` 将传入的参数值存入数组。当以这种方式访问实例变量时，前缀 @ 是必须的——否则变量虽可创建但无法从 Ruby 内引用。

#### 21.1.1 构建我们的扩展

遵循下面的步骤：


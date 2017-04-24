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

* 在 C 源文件 `my_test.c` 的同一目录下，创建一个名为 `extconf.rb` 的文件，包括以下两行：

```ruby
require "mkmf"
create_makefile("my_test")
```

* 运行 `extconf.rb`，这会生成一个 Makefile
* 使用 make 来构建扩展。

构建这个扩展的所有结果，会被组装成一个共享库（一个 `.so`、一个 `.dll` 或 `.bundle`）

#### 21.1.2 运行我们的扩展

可以简单地通过在运行时动态 require 来使用它。

```ruby
require "my_test"
require "test/unit"
# xxx
```

在看到扩展可以正常工作后，然后我们可以通过运行 `make install` 将它在全局安装。

### 21.2 C 中的 Ruby 对象

在操作 Ruby 对象之前，我们需要找出如何从 C 表示和访问 Ruby 的数据对象。

大部分 Ruby 对象，被表示为一个 C 指针，它指向了包括对象数据和其他实现细节的一块内存区域。在 C 代码中，所有这些引用都是 VALUE 类型的变量，因此当你传递 Ruby 对象时，所传递的是 VALUE。

出于性能的原因，Ruby 将 `Fixnum`、`Symbol`、`true`、`false` 和 `nil` 实现为所谓的立即值（immediate value）。它们的值也保存在 VALUE 类型的变量中，但是它们不是指针。相反地，它们的值直接保存在变量中。

因此，某些时候 VALUE 是指针，而某些时候它们是直接值。

所有指向某个内存区域的指针，都是 4 或 8 字节边界对齐的。这意味着我们可以确保指指针的低两位总是 0。当它想要储存一个立即值时，它会安排至少将其中一位设置为 1，其余的解释器代码可以将指针和直接值区别开来。

这是 Ruby 如何在 C 中实现面向对象的代码：Ruby 对象是内存中分配的一个结构，包括了实例变量的表（table）和有关这个类的信息。类本身是另一个对象（也是内存中分配的结构），包括了类的实例方法表。Ruby 是在此基础上建立的。

#### 21.2.1 操作直接对象

Fixnum 的值是一个 31 位的数字，它将原来的数字左移一位，并设置 LSB（least significant bit，最低有效位，bit 0）为 1。当 VALUE 被作为一个指针指向某个具体的 Ruby 结构时，总能确保 LSB 的值为 0；其他直接值的 LSB 也为 0。因此，一个简单的位测试可以告诉你它是否是一个 Fixnum。`FIXNUM_P` 的宏包装了这个测试：

```c
FIXNUM_P(value)	// 如果 value 是 Fixnum，则非 0
SYMBOL_P(value)	// 如果 value 是 Symbol，则非 0
NIL_P(value)	// 如果 value 是 nil，则非 0
RTEST(value)	// 如果 value 是 nil 或 false，则非 0
```

其他的直接值（true、false 和 nil）在 C 中被分别表示为 Qtrue、Qfalse 和 Qnil 的常量。你可以直接测试 VALUE 变量，或者使用转换宏（由它进行适当的转型）

#### 21.2.2 操作字符串

在 C 中，使用 `null` 结尾的字符串。不过，Ruby 字符串更一般化，可能包括若干内嵌的 null。因此，操作 Ruby 字符串最安全的方式是，按照解释器所做的那样，同时使用指针和长度。实际上，Ruby String 对象是 RString 结构的一个引用，而 RString 结构包括指针和长度成员。你可以通过 RSTRING 宏来访问这个结构。

```c++
VALUE str;
RSTRING(str)->len;	// Ruby 字符串的长度
RSTRING(str)->ptr;	// 字符串的存储指针
```

当你需要一个字符串值的时候，与其直接使用 VALUE 对象，还不如调用 StringValue，并以原来的值作为参数。它会返回一个对象，你可以对其使用 RSTRING 宏，或者如果它无法从原来的值得到一个字符串时，则抛出异常。

StringValue 方法检查其操作数是否是一个 String，如果不是，则它尝试调用对象的 `to_str`，如果调用失败，则抛出一个 TypeError 异常。

因此，如果你想编写迭代 String 对象中所有字符的代码，可以这样编写：

```c++
static VALUE iterate_over(VALUE original_str) {
  int i;
  char *p;
  VALUE str = StringValue(original_str);
  p = RSTRING(str)->ptr;	// may be null
  for (i = 0; i < RSTRING(str)->len; ++i, ++p) {
    // process *p
  }
  return str;
}
```

如果你想跳过长度信息，而只访问底层的字符串指针，可以使用便捷方法 StringValuePtr，它得到字符串的引用，然后返回指向其内容的 C 指针。

如果你想使用字符串来访问或控制某些外部资源，那么你可能希望钩入（hook） Ruby 的 tainting 机制中。

#### 21.2.3 操作其他对象

当 VALUE 不是直接对象时，它们指向定义了 Ruby 对象结构的指针——你不能让 VALUE 指向任意内存区域。基本內建类的结构在 `ruby.h` 中定义，命名为 RClassname。

你有多种方式可以检查某个特定的 VALUE 是何种类型的结构。宏 TYPE(obj) 会返回一个常量，表示给定对象的 C 类型：`T_OBJECT`、`T_STRING` 等。内建类的常量在 `ruby.h` 中定义。注意，这里所说的 `type` 是实现中的细节——和对象的类不同。

如果你想确保 VALUE 指针指向一个特定的结构，可以使用宏 `Check_Type`，如果 value 不是预期的 type（`T_STRING`、`T_FLOAT` 这样的常量），则它会抛出一个 TypeError 异常。

```c++
Check_Type(VALUE value, int type)
```

内建类的 class 对象保存在名为 `rb_cClassname` 的 C 全局变量中，模块的命名为 `rb_mModulename`。

>   Ruby 1.8 之前的字符串访问
>
>   在 Ruby 1.8 之前，如果认为 VALUE 包含一个字符串，那么你直接访问 RSTRING 成员即可。然而，在 1.8 中，随着 duck typing 以及各种优化的逐步引入，这种方法可能无法入往常那样工作。特别地，STRING 对象的 ptr 成员可能用 null 来表示长度为 0 的字符串。如果你使用 1.8 中的 StringValue 方法，则它会处理这种情况，将 null 指针重新指向一个唯一的、共享的空字符串。
>
>   要编写一个在 Ruby 1.6 和 1.8 中都可以工作的扩展：
>
>   ```c++
>   #if !defined(StringValue)
>   # define StringValue(x) (x)
>   #endif
>   #if !defined(StringValuePtr)
>   # define StringValuePtr(x) ((STR2CSTR(x)))
>   #endif
>   ```
>
>   如果希望你的代码即使在 1.6 中运行，也具有 1.8 中的 duck-typing 的行为，那么你可能希望稍有不同地定义 StringValue:
>
>   ```c++
>   #if !defined(StringValue)
>   # define StringValue(x) do {
>     if (TYPE(x) != T_STRING) x = rb_str_to_str(x);
>   } while (0)
>   #endif
>   ```

直接更改这些 C 结构中的数据是不可取的，不过——你可以查看，但不要触动它们。

要解引用（dereference）这些 C 结构的成员，你必须将一般化的 VALUE 转型为适当的结构类型。`ruby.h` 包含了许多宏为你执行适当的转型，让你可以很容易地对结构成员解引用。这些宏的名称为 `RCLASSNAME`，例如 `RSTRING`。

```c++
VALUE arr;
RARRAY(arr)->len;
RARRAY(arr)->capa;
RARRAY(arr)->ptr;
```

对散列表（RHASH）、文件（RFILE）等也有类似的访问方法。

在扩展代码中不要过于依赖对类型的检查。

#### 21.2.4 全局变量

多数时候，由你的扩展实现类，而 Ruby 代码使用这些类。你在 Ruby 和 C 代码中共享的数据，会整洁地包装（wrap）到类的对象中。

有时候你希望实现全局变量，最简单的方法是，让变量作为一个 VALUE（也就是说，一个 Ruby 对象）。然后你可以将 C 变量的地址绑定到一个 Ruby 变量的名字。在这种情况下，`$` 前缀是可选的，但是它可以帮助你澄清这是一个全局变量。注意一点，让栈中的变量作为 Ruby 的全局变量，不会（长期）有效。

```c++
static VALUE hardware_list;
static VALUE Init_SysInfo() {
  rb_define_class(....);
  hardware_list = rb_ary_new();
  rb_define_variable("$hardware", &hardware_list);
  // ...
  rb_ary_push(hardware_list, rb_str_new2("DVD"));
  rb_ary_push(hardware_list, rb_str_new2("CDPlayer1"));
  rb_ary_push(hardware_list, rb_str_new2("CDPlayer2"));
}
```

Ruby 端可以用 `$hardware` 来访问 C 变量 `hardware_list`。

也许你要定义的全局变量，它的值在访问时必须通过计算才能得到。你可以通过定义 `hooked` 和虚拟（virtual）变量来完成。`hooked` 变量是一个真实的变量，当访问到对应的 Ruby 变量时，由一个具名（named）的函数初始化。虚拟变量也有些类似，但并没有真正的存储：它们的值来自于对 hook 函数的求值。

如果你从 C 创建一个 Ruby 对象，把它保存在一个 C 全局变量中，而不将它暴露给 Ruby，你至少要将它告知给垃圾收集器，以免疏忽而忘记回收。

```c++
static VALUE obj;
// ...
obj = rb_ary_new();
rb_global_variable(obj);
```

### 21.3 Jukebox 扩展

#### 21.3.1 包装 C 结构

任何时候，你想要如 Ruby 对象那样来处理一个 C 结构，你需要将它包装（wrap）成一个特殊的 Ruby 内部类，称为 DATA（类型为 `T_DATA`）。有两个宏完成这一包装，其中一个宏会再次返回你的结构。

#### 21.3.2 API：C 数据类型的包装

```c++
VALUE Data_Wrap_Struct(VALUE class, void (*mark)(), void (*free)(), void *ptr)
```

包装给定的 C 数据类型 ptr，注册两个垃圾回收例程，并返回一个指向实际 Ruby 对象的 VALUE 指针。返回对象的 C 类型是 `T_DATA`，且它的 Ruby 类是 class。

```c++
VALUE Data_Make_Struct(VALUE class, c-type, void (*mark)(), void (*free)(), c-type *)
```

分配指定类型的结构体并清零，然后继续 `Data_Wrap_Struct` 的工作。`c-type` 是你要包装的 C 数据类型的名字，而不是类型的变量。

```c++
Data_Get_Struct(VALUE obj, c-type, c-type *)
```

返回原本的指针。这个宏是对宏 `DATA_PTR(obj)` 的类型安全的包装，它会对指针进行评估。

`Data_Wrap_Struct` 创建的对象是普通的 Ruby 对象，唯一不同的是它有额外的、无法从 Ruby 中访问的 C 数据类型。C 数据类型和对象所包含的实例变量是分开的。

Ruby 使用标记和清扫（mark and sweep）的垃圾回收方式。在标记阶段，Ruby 查找指向内存区域的指针。它标记这些区域为“使用中”。如果这些区域本身包括更多的指针，这些指针所指向的内存也会被标记，以此类推。在标记的最后阶段，所有被引用的内存已经被标记了，而所有孤立的区域则不会有标记。当清扫阶段开始时，释放那些没有被标记的内存。

为了参与到 Ruby 标记后清扫的垃圾回收过程，你必须定义一个例程来释放你的结构，或者还需要一个例程来完成从你的结构到其他结构的引用标记。两个例程都接收 void 指针，它指向了你的结构。mark 例程在垃圾回收器处于标记阶段被调用。如果你的结构引用了其他的 Ruby 对象，那么你的标记函数需要用 `rb_gc_mark(value)` 来标识出这些对象。如果结构没有引用其他的 Ruby 对象，你可以简单地以 0 作为函数的指针。

当对象需要清除时，垃圾收集器将调用 `free` 例程来释放它。如果你自己分配了某些内存（例如，使用 `Data_Make_Struct`），你需要传入一个释放函数——即便它就是标准 C 库中的 free 例程。对你分配的复杂结构，你的释放函数可能需要遍历这个结构来释放所有分配的内存。

#### 21.3.3 对象创建

Ruby 1.8 完善了对象的创建和初始化。虽然旧有的方式也可以工作，但是使用分配函数的新方式，要整洁得多。

我们需要实现一个分配函数以及一个初始化方法。

#### 21.3.4 分配函数

分配函数负责创建对象使用的内存。如果你所实现的对象，并不需要除 Ruby 实例变量之外的数据，那么你无须编写一个分配函数——Ruby 默认的分配器就可以满足了。但是如果你的类包装了一个 C 结构，你需要在分配函数中为这个结构分配空间。分配函数将要分配对象的类（class）作为参数。

```c++
static VALUE cd_alloc(VALUE klass){
  CDJukebox *jukebox;
  VALUE obj;
  
  jukebox = new_jukebox();
  obj = Data_Wrap_Struct(klass, 0, cd_free, jukebox);
  
  return obj;
}
```

你还需要在类的初始化代码中注册你的分配函数。

```c++
void Init_CDPlayer() {
  cCDPlayer = rb_define_class("CDPlayer", rb_cObject);
  rb_define_alloc_func(cCDPlayer, cd_alloc);
  // ...
}
```

大部分对象可能也需要定义一个初始化函数。分配函数创建一个空的、未初始化的对象，而我们需要填入具体的值。

```c++
static VALUE cd_initialize(VALUE self, VALUE unit) {
  int unit_id;
  CDJukebox *jb;
  Data_Get_Struct(self, CDJukebox, jb);
  unit_id = NUM2INT(unit);
  assign_jukebox(jb, unit_id);
  return self;
}
```

这种多步对象创建协议的原因是，让解释器能够处理必须以 “后门方式” 创建对象的情况。一个例子是，当对象要从列集的（marshaled）形式中反序列化时。此时，解释器需要创建一个空的对象（通过调用分配器），但是它不会调用初始化函数（因为它不知道要使用的参数）。另一个常见的情况是，当对象被复制或克隆时。

因为用户可以选择绕过构造函数，你需要确保分配代码使返回的对象处于有效状态。它可能并没有包括曾经由 `#initialize` 设置的全部信息，但是至少应该是可用的。

#### 21.3.5 克隆对象

所有 Ruby 对象都可以使用 dup 或 clone 方法来拷贝。两者都会通过调用分配函数产生调用类的一个新实例。然后，它们从原来的对象中拷贝实例变量。`clone` 会拷贝得更深入些，它会拷贝原来对象的单体类（singleton class）和标志（例如指示对象被冻结的标志）。你可以认为 dup 是内容的拷贝，而 clone 是整个对象的拷贝。

不过，Ruby 解释器并不知道如何处理你编写的 C 扩展中对象的内部状态。

>   1.8 版本之前的对象分配
>
>   在 Ruby 1.8 之前，如果你想要在对象中分配额外的空间，要么你将代码放到 initialize 方法中，要么你必须为你的类定义一个 new 方法。建议使用下面的混合方法，来做到 1.6 和 1.8 扩展间最大的兼容。
>
>   ```c++
>   static VALUE cd_alloc(VALUE klass) {
>     // same as before
>   }
>   static VALUE cd_new(int argc, VALUE *argv, VALUE klass) {
>     VALUE obj = rb_funcall2(klass), rb_intern("allocate"), 0, 0);
>     rb_obj_call_init(obj, argc, argv);
>     return obj;
>   }
>   void init_CDPlayer() {
>     // ...
>   #if HAVE_RB_DEFINE_ALLOC_FUNC
>     // 1.8 allocation
>     rb_define_alloc_func(cCDPlayer, cd_alloc);
>   #else
>     // define manual allocation function for 1.6
>     rb_define_singleton_method(cCDPlayer, "allocate", cd_alloc, 0);
>   #endif
>     rb_define_singleton_method(cCDPlayer, "new", cd_new, -1);
>     // ...
>   }
>   ```
>
>   不过，你可能还需要处理克隆和复制，并考虑当你的对象被列集时会如何。

为了处理它，解释器将拷贝对象内部状态的职责委托给你自己的代码。在拷贝对象的实例变量之后，解释器调用新对象的 `initialize_copy` 方法，传入原对象的引用。

#### 21.3.6 集成到一起

略

### 21.4 内存分配

有时你可能需要在扩展中分配内存，但不用于对象的存储——也许你已经得到了 Bloom 滤波器（Bloom filter）一个巨大的位图（bitmap）或者 Ruby 并不直接使用的许多小的数据结构。

为了让垃圾收集器正确地工作，应该使用下面的内存分配例程。这些例程比标准的 `malloc` 稍稍多做了些工作。例如，如果 `ALLOC_N` 判断它无法分配得到所需数量的内存，将会调用垃圾收集器来尝试收回一些空间。如果无法分配或者所需数量的内存不可获得，它将引发一个 `NoMemError` 异常。

#### 21.4.1 API：内存分配

```c++
type * ALLOC_N(c-type, n)	// 分配 n 个 c-type 的对象，其中 c-type 是 C 类型的字面名称，而非该类型的变量
type * ALLOC(c-type)		// 分配一个 c-type，并且将结果转型为该类型的指针
REALLOC_N(var, c-type, n)	// 重新分配 n 个 c-type，并将结果复制给指针 var，它指向类型为 c-type 的变量
type * ALLOCA_N(c-type, n)	// 在栈上分配 c-type 的 n 个对象——这部分内存会在调用 ALLOCA_N 的函数返回时自动释放
```

### 21.5 Ruby 的类型系统

在 Ruby 中，我们很少依赖于一个对象的类型（或 class），而更多地关注于它的能力，这被称为 `duck typing`。

作为一个扩展的编写者，应该尝试避免检查传递给你的参数类型。看看是否有类似 `rb_check_xxx_type` 的方法帮你将参数转换到你需要的类型。如果没有，查找现有的某个可以帮助你进行转换的函数（例如 `rb_Array` ）。如果实现的某些对象可以有意义地用作一个 Ruby 字符串或数组，考虑实现 `to_str` 或 `to_ary` 方法，允许扩展所实现的对象被用于字符串或数组的上下文中。

### 21.6 创建一个扩展

基本的步骤：

*   在指定的目录创建 C 源文件
*   在 `lib` 子目录中创建所有 Ruby 的支持文件（可选）
*   创建 `extconf.rb`
*   运行 `extconf.rb` 来为目录中的 C 文件创建 `Makefile`
*   运行 `make`
*   运行 `make install`

#### 21.6.1 通过 `extconf.rb` 创建 `Makefile`

构建扩展的总体流程，关键是，你作为开发者所创建的 `extconf.rb` 程序。在 `extconf.rb` 中，你编写一个简单的程序来判断在用户系统中有哪些特性可用，以及这些特性位于何处。执行 `extconf.rb` 会建立一个定制的 `Makefile`，它是根据你的应用和编译时所用的系统来裁剪的。当你对该 Makefile 运行 make 命令时，你的扩展被构建并（可选地）安装。

最简单的 `extconf.rb` 可能只有两行，而对许多扩展来说已经足够了。

```ruby
require "mkmf"
create_makefile("Test")
```

第一行引入了 mkmf 库模块。它包括了所有我们要使用的命令。第二行位名为 "Test" 的扩展创建一个 Makefile（makefile 的名字总是 Makefile）。Test 将会从当前目录中的 C 源文件构建，当你的代码被加载时，Ruby 调用它的 `Init_Test`  方法。

假设在只有一个源文件（main.c）的目录中执行 `extconf.rb` 程序，其结果是用来构建我们扩展的一个 `makefile`。在 Linux 机器上，它执行下面的命令。

```shell
gcc -fPIC -I/usr/local/lib/ruby/1.8/i686-linux -g -02 -c main.c -o main.o
gcc -shared -o Test.so main.o -lc
```

编译的结果是 `Test.so`，可以通过 `require` 在运行时动态地链接到 Ruby 中。

如果你的扩展需要默认默认编译环境所没有的某些头文件或库，或者你根据某些库或函数的存在与否来进行条件编译，你还需要多做些工作。

一个常见的需求是指定查找头文件和库的非标准目录。首先，你的 `extconf.rb` 应该包括一个或多个 `dir_config` 命令。这指定了一个目录集合的标记。然后，当你运行 `extconf.rb` 程序时，告诉 mkmf 在当前系统中对应的物理目录在什么地方。

>   划分命名空间
>
>   与其将扩展编写者的工作直接安装到 Ruby 的某个库目录中，还不如使用子目录将文件组织起来。这对 `extconf.rb` 很简单，如果 `create_makefile` 调用的参数包括斜线，那么 `mkmf` 会认为在最后一个斜线之前的是目录名，而余下的是扩展的名字。扩展会被安装到指定的目录中（相对 Ruby 的目录树）。
>
>   ```ruby
>   require "mkmf"
>   create_makefile("wibble/Test")
>   ```
>
>   然而，当需要这个类时：
>
>   ```ruby
>   require "wibble/Test"
>   ```

如果 `extconf.rb` 包括 `dir_config(name)` 的代码行，你可以用命令行选项给出对应目录的位置。

```c++
--with-name-include=directory	// 将 directory/include 添加到编译器的选项中
--with-name-lib=directory		// 将 directory/lib 添加到链接器的选项中
--with-name-dir=directory		// 将 directory/lib 和 directory/include 分别添加到链接器和编译器的选项中
```

你也可以使用为你的机器构建 Ruby 时所使用的 `--with` 选项。这意味着，你可以发现并使用 Ruby 本身使用库的位置。

```shell
% ruby extconf.rb=--with-cdjukebox-dir=/usr/local/cdjb
```

产生的 Makefile 会认为 `/usr/local/cdjb/lib` 含有库文件，而 `/usr/local/cdjb/include` 含有头文件。

`dir_config` 命令添加用于搜索库和头文件的位置列表。但它并不会将这些库链接到你的应用。

`have_library` 查找某个具名库的指定入口点。如果它找到这个入口点，便将这个库添加到链接扩展所使用库的列表中。`find_library` 与之类似，它允许你指定一个目录列表来搜索库。

某个特定库的位置可能会因主机系统的不同而不同。

标准的 Ruby 发布，它只尝试编译你的系统所支持的那些扩展。

`have_header` 查找头文件；`have_func` 检查目标环境中的所有库是否支持某个特定的函数。

如果 `have_header` 或 `have_func` 找到了目标，它们均会定义预处理器的常量。名字的形式是，将目标名转换为大写字母且在前面加上 `HAVE_`。你的 C 代码则可以利用这一宏定义，例如：

```c++
#if defined(HAVE_HP_MP3_H)
# include <hp_mp3.h>
#endif
#if defined(HAVE_SETPRIORITY)
 err = setpriority(PRIOR_PROCESS, 0, -10)
#endif
```

如果你有特殊的需要，而这些 mkmf 的命令无法满足时，你的程序可以直接向全局变量 `$CFLAGS` 和 `$LFLAGS` 中添加选项，这些选项会被分别传入到编译器和链接器。

##### 安装目标

你的 `extconf.rb` 所产生的 Makefile，会包括一个 `install` 目标。它会将你的共享库对象正确拷贝到你（或你用户）的本地文件系统中。目标与你运行 `extconf.rb` 时使用的 Ruby 解释器的安装位置有关。如果你的系统中安装了多个 Ruby 解释器，你的扩展会被安装到那个运行 `extconf.rb` 的 Ruby 解释器的目录树中。

除了安装共享库之外，`extconf.rb` 还会查看是否存在 `lib/` 子目录。如果有，它会将这些 Ruby 文件随着共享库一同安装。如果你将编写扩展的工作分成两部分，包括底层的 C 代码和高层的 Ruby 代码时，这很有用。

#### 21.6.2 静态链接

如果你的系统不支持动态链接，或者如果你有一个扩展模块希望静态链接到 Ruby 本身，编辑 Ruby 发布中的 `ext/Setup` 将扩展的目录列表添加到该文件中。在扩展的目录中，创建一个名为 `MANIFEST` 的文件，其中包括扩展的所有文件（源文件、`extconf.rb`、`lib/` 等等）。然后重新构建 Ruby。Setup 中列出的扩展将会被静态地链接到 Ruby 可执行程序中。如果你想禁止所有的动态链接，并静态链接所有的扩展，编辑 `ext/Setup` 加入下面的选项：

```shell
option nodynamic
```

#### 21.6.3 一个捷径

如果你希望扩展一个现有的用 C 或 C++ 编写的库，你可以调查一下 SWIG。SWIG 是一个接口生成器：它使用库的定义（通常从头文件获得）并自动生成从其他语言访问这个库的粘合（glue）代码。SWIG 支持 Ruby，意味着它可以生成将外部库包装到（wrap）Ruby 类的 C 源文件。

### 21.7 内嵌 Ruby 解释器

除了通过添加 C 代码来扩展 Ruby 外，你还可以把 Ruby 嵌入到你的应用中。你有两种方式来这么做。一种是通过调用 `ruby_run` 来让解释器得到控制权，但这样解释器永远不会从 `ruby_run` 调用中返回。

```c++
#include "ruby.h"
int main(void) {
  ruby_init();
  ruby_init_loadpath();
  ruby_script("embedded")
  rb_load_file("start.rb");
  ruby_run();
  exit(0);
}
```

为了初始化 Ruby 解释器，还可以调用 `ruby_init()`。但是在某些平台上，你需要在此之前施行一些特殊的步骤。

```c++
#if defined(NT)
 NTInitialize(&argc, &argv);
#endif
#if defined(__MACOS__) && defined(__MWERKS__)
  argc = ccommand(&argv);
#endif
```

查看 Ruby 发布中的 `main.c`，看看你的平台需要哪些特殊的宏定义或设置。

你需要 Ruby 的头文件和库文件来编译这些嵌入的代码。

第二种嵌入 Ruby 的方式是，允许 Ruby 代码和你的 C 代码交互：C 代码调用 Ruby 代码，接着 Ruby 代码作出响应。你和寻常一样初始化解释器。然后，不再进入解释器的主循环，相反你调用 Ruby 代码中的特定方法。当这些方法返回时，你的 C 代码则取回控制权。

不过，如果 Ruby 代码引发了一个异常而没有被捕捉，那么你的 C 程序会退出。你需要像解释器那样保护所有可能异常的调用。

`rb_protect` 方法可以包装对另一个 C 函数的调用，后者应该调用我们的 Ruby 方法。然而，`rb_protoct` 包装的方法被定义为只能接受一个参数。要传入多个参数将牵扯到某些丑陋的 C 转型。

编写一个 C 程序调用类的实例：要创建它的实例，需要得到类对象（通过查找一个顶层的常量，其名字亦是类的名字）。然后可以让 Ruby 创建这个类的实例——实际上 `rb_class_new_instance` 等同于 `Class.new`。（开头两个为 0 的参数是变量数和指针变量本身的空指针。）一旦我们得到这个对象，就可以使用 `rb_funcall` 调用实例方法。

Ruby 解释器并非为嵌入到其他语言而编写的。最大的问题可能是它在全局变量中维护其状态，因此它不是线程安全的。你可以嵌入 Ruby——每个进程只有一个解释器。

#### 21.7.1 API：嵌入 Ruby API

略

### 21.8 将 Ruby 连接到其他语言

你可以用任何语言来编写扩展，只要能将两个语言用 C 连接起来。

另外，你可以使用中间件（例如 SOAP 或 COM）将 Ruby 连接到其他语言。

### 21.9 Ruby C 语言 API

有一些 C 级别的函数，你可能发现在编写扩展时它们非常有用。

某些函数需要一个 ID：你可以使用 `rb_intern` 得到某个字符串对应的 ID，并使用 `rb_id2name` 从 ID 重建这个字符串名字。

大部分 C 函数在 Ruby 中有对等的方法。

#### 21.9.1 API：定义类

略

#### 21.9.2 API：定义结构

略

#### 21.9.3 API：定义方法

略

#### 21.9.4 API：定义变量和常量

略

#### 21.9.5 API：调用方法

略

#### 21.9.6 API：异常

略

#### 21.9.7 API：迭代器

略

#### 21.9.8 API：访问变量

略

#### 21.9.9 API：对象状态

略

#### 21.9.10 API：常用的方法

略
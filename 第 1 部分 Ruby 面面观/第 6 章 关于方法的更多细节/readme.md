### 6.1 定义一个方法

可以使用关键字 `def` 来定义一个方法。方法名必须以一个小写字母开头。表示查询的方法名通常以 ? 结尾。会修改接收者对向（receiver）的方法，可以用 ! 结尾。可以被赋值的方法以一个等号（=）结尾。

方法参数两边的括号是可选的；惯例是，当方法有参数时则使用括号，否则即忽略它们：

```ruby
def my_new_method(arg1, arg2, arg3)
  # Code for the method would go here
end

def my_other_new_method
  # Code for the method would go here
end
```

Ruby 可以让你指定方法参数（argument，实参）的默认值：

```ruby
def cool_dude(arg1="Miles", arg2="Coltrane", arg3="Roach")
  "#{arg1}, #{arg2}, #{arg3}."
end
```

方法体内是普通的 Ruby 表达式，你不能在方法内定义非单件（nonsingleton）类或模块。如果你在一个方法内定义另一个方法，内部的方法只有在外部方法执行时猜得到定义。

#### 6.1.1 可变长度的参数列表

在”普通“的参数名前放置一个星号（*）即可

#### 6.1.2 方法和 Block

当调用一个方法时，可以用一个 block 与之相关联。通常，你可以使用 `yield` 从方法内部调用这个 block。

不过，如果方法定义的最后一个参数前缀为 &，那么所关联的 Block 会被转化为一个 Proc 对象，然后赋值给这个参数。

### 6.2 调用方法

如果你省略了接收者，其默认为 self，也就是当前的对象：

```ruby
self.class		# ->	Object
self.frozen?	# ->	false
frozen?  		# ->	false
self.object_id	# ->	978140
object_id		# ->	978140
```

早先的 Ruby 版本允许你在方法名和左括号之间放置空格，使这个问题更为严重。这使得解析变得困难：括号是参数列表的起始呢，还是表达式的起始？在 Ruby 1.8 中，如果你在方法名和左括号之间置入空格，你会得到一个警告。

#### 6.2.1 方法返回值

如果你给 return 多个参数，方法会将它们以数组的形式返回。你可以使用并行赋值来收集返回值。

##### 在方法调用中的数组展开

当你调用一个方法时，你可以分解一个数组，这样每个成员都被视为单独的函数。在数组参数（必须再所有普通参数的后面）前加一个星号就可以完成这一点。

##### 让 block 更加动态

一个例子，将完成计算的 block 抽取出来：

```ruby
times = gets
number = Integer(gets)
if times =~ /^t/
  puts((1..10).collect {|n| n*number }.join(", "))
else
  puts((1..10).collect {|n| n+number}.join(", "))
end
```

 以下将 block 抽取出来：

```ruby
times = gets
number = Integer(gets)
if times =~ /^t/
  calc = lambda {|n| n * number}
else
  calc = lambda {|n| n + number}
end
puts((1..10).collect(&calc).join(", "))
```

如果方法的最后一个参数前有 & 符号，Ruby 将认为它是一个 Proc 对象。它会将其从参数列表中删除，并将 Proc 对象转换为一个 block，然后关联到该方法。

##### 收集散列参数

Ruby 1.8 并不支持关键字参数

```ruby
class SongList
  def create_search(name, params)
    # ...
  end
end
list.create_search("short jazz songs",
  {
    "genre"	=>	"jazz",
    "duration_less_than"	=>	270
  })
```

第一个参数是搜索的名字，第二个是一个散列式，其中包括搜索的参数。使用散列表意味着我们可以模拟关键字。不过，这种方式有些笨重，并且一组大括号容易被误写为一个和方法关联的 block。因此，Ruby 提供了一种快捷方式。只要在参数列表中，散列数组在正常的参数之后，并位于任何数组或 block 参数之前，你就可以只使用 `key => value` 对。所有这些对会被集合到一个散列数组中，并作为方法的一个参数传入。无须使用大括号。

```ruby
list.create_search("short jazz songs",
  "genre" => "jazz",
  "duration_less_than" => 270)
```

最后，Ruby 的一种惯用技法是，可以使用符号（symbol）而非字符串作为参数，符号清楚地表达了你所引用的某个事物的名字。

```ruby
list.create_search("short jazz songs",
  :genre => :jazz,
  :duration_less_than => 270)
```
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
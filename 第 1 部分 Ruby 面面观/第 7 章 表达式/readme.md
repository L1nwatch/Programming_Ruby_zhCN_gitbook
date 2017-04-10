Ruby 和其他语言的一个不同之处就是任何东西都能返回一个值：几乎所有东西都是表达式。

明显的一个好处是能够实现链式语句。

```ruby
a = b = c = 0				# ->	0
[3, 1, 7, 0].sort.reverse	# ->	[7, 3, 1, 0]
```

不太明显的好处是，C 和 Java 中的普通语句在 Ruby 中也是表达式。例如，if 和 case 语句都返回最后执行的表达式的值。

### 7.1 运算符表达式

实际上，Ruby 中的许多运算符都是由方法调用来实现的。例如，当你执行 `a*b+c` 时，等价于：

```ruby
(a.*(b)).+(c)
```

你可以重新定义任何不满足你需求的基本算术方法：

```ruby
class Fixnum
  alias old_plus +
    def +(ohter)
      old_plus(other).succ
    end
end
```

更有用的是，你写的类可以像內建对象那样参数到运算符表达式中：

```ruby
class Songe
  def [](from_time, to_time)
    # ...
  end
end
```

### 7.2 表达式之杂项


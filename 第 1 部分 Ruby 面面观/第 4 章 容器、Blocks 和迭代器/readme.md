容器（containers），是指含有一个或多个对象引用的对象

### 4.1 容器

#### 4.1.1 数组

数组由 `[ ]` 操作符来进行索引。和 Ruby 的大多数操作符一样，它实际上是一个方法，因此可以被子类重载。

也可以使用一对数字 `[start, count]` 来访问数组。这将返回一个包含从 start 开始的 count 个对象引用的新数组。

```ruby
a[1, 3]	# ->	[3, 5, 6]
```

还可以使用 range 来对数组进行索引，其开始和结束位置被两个或 3 个点分割开。两个点的形式包含结束位置，而 3 个点的形式不包含。

```ruby
a[1..3]	# ->	[3, 5, 6]
```

`[ ]` 操作符有一个相应的 `[ ]=` 操作符，它可以设置数组中的元素。如果下标是单个整数，那么其位置上的元素将被赋值语句右边的东西所替换。造成的任何间隙将由 nil 来填充。

如果 `[ ]=` 的下标是两个数字（起点和长度）或者是 range，那么原数组中的那些元素将被赋值语句右边的东西所替换。如果长度是 0，那么赋值语句右边的东西将被插入到数组的起点位置之前，且不会删除任何元素。如果右边本身是一个数组，那么其元素将替换掉原数组对应位置上的东西。如果索引下表选择的元素个数和赋值语句右边的元素个数不一致，那么数组会自动调整其大小。

```ruby
a = [1, 3, 5, 7, 9]	# ->	[1, 3, 5, 7, 9]
a[2, 2]	= 'cat'		# ->	[1, 3, "cat", 9]
a[2, 0] = 'dog'		# ->	[1, 3, "dog", "cat", 9]
a[1, 1]	= [9, 8, 7]	# ->	[1, 9, 8, 7, "dog", "cat", 9]
a[0..3]	= []		# ->	["dog", "cat", 9]
a[5..6] = 99, 98	# ->	["dog", "cat", 9, nil, nil, 99, 98]
```

#### 4.1.2 散列表

#### 4.1.3 实现一个 SongList 容器

`SongList#append` 方法将给定的歌曲添加到 `@songs` 数组的尾部。它会返回 self，即当前 SongList 对象的引用。这样可以让我们把对 append 的多个调用链接在一起。

```ruby
class SongList
  def initialize
    @songs = Array.new
  end
  def append(song)
    @songs.push(song)	# 往队列末尾添加元素
    self
  end
  def delete_first
    @songs.shift		# 删除队列开头的元素
  end
  def [](index)
    @songs[index]
  end
end
```

Ruby 标准发行版自带一个称为 TestUnit 的测试框架。测试需要必要的初始化，以告诉 Ruby 使用 TestUnit 测试框架。

```ruby
require "test/unit"

class TestSongList < Test::Unit::TestCase
  def test_delete
    list = songList.new
    s1 = Song.new("title1", "artist1", 1)
    s2 = Song.new("title2", "artist2", 2)
    s3 = Song.new("title3", "artist3", 3)
    s4 = Song.new("title4", "artist4", 4)
    
    list.append(s1).append(s2).append(s3).append(s4)
    
    assert_equal(s1, list[0])
    assert_equal(s3, list[2])
    assert_nil(list[9])
    
    assert_equal(s1, list.delete_first)
    assert_equal(s2, list.delete_first)
    assert_equal(s4, list.delete_last)
    assert_equal(s3, list.delete_last)
    assert_nil(list.delete_last)
  end
end
```

### 4.2 Blocks 和迭代器

要实现查找功能，可以通过遍历该数组的所有元素，并查找出匹配的元素：

```ruby
class SongList
  def with_title(title)
    for i in 0...@songs.length
      return @songs[i] if title == @songs[i].name
    end
    return nil
  end
end
```

在某种程度上，for 循环和数组的耦合过于紧密：需要知道数组的长度，然后依次获得其元素的值，直到找到一个匹配为止。以下这种方式也许会更好些：

```ruby
class SongList
  def with_title(title)
    @songs.find {|song| title == song.name }
  end
end
```

#### 4.2.1 实现迭代器

block 在代码中只和方法调用一起出现；block 和方法调用的最后一个参数处于同一行，并紧跟在其后（或者参数列表的右括号的后面）。其次，在遇到 block 的时候并不立刻执行其中的代码。Ruby 会记住 block 出现时的上下文（局部变量、当前对象等）然后执行方法调用。

block 也可以返回值给方法。block 内执行的最后一条表达式的值被作为 yield 的值返回给方法。

```ruby
class Array
  def find
    for i in 0...size
      value = self[i]
      return value if yield(value)
    end
    return nil
  end
end

[1， 3， 5， 7， 9].find {|v| v * v > 30 }	# ->	7
```

一个常用的迭代器是 collect，它从收集中获得各个元素并传递给 block。block 返回的结果被用来生成一个新的数组，例如：

```ruby
["H", "A", "L"].collect {|x| x.succ }	# ->	["I", "B", "M"]
```

下面的例子使用了 `do...end` 来定义 block。这种方式定义 block 和使用花括号定义 block 的唯一区别是优先级：`do...end` 的绑定低于 `{...}`

```ruby
f = File.open("testfile")
f.each do | line|
  puts line
end
f.close
```

`inject` 方法让你可以遍历收集的所有成员以累计出一个值。例如：

```ruby
[1, 3, 5, 7].inject(0) {|sum, element| sum + element}			# ->	16
[1, 3, 5, 7].inject(1) {|product, element| product * element}	# ->	105
```

`inject` 是这样工作的：block 第一次被执行时，sum 被置为 inject 的参数，而 element 被置为收集的第一个元素。接下来每次执行 block 时，sum 被置为上次 block 被调用时的返回值。inject 的最后结果是最后一次调用 block 返回的值。还有一个技巧：如果 inject 没有参数，那么它使用收集的第一个元素作为初始值，并从第二个元素开始迭代：

```ruby
[1, 3, 5, 7].inject {|sum, element| sum + element}				# ->	16
[1, 3, 5, 7].inject {|product, element| product * element}		# ->	105
```

##### 内迭代器和外迭代器

在 Ruby 中，迭代器集成于收集内部——它只不过是一个方法，和其他方法不同的是，每当产生新的值的时候调用 yield。使用迭代器的不过是和该方法相关联的一个代码 block 而已。

在其他语言中，收集本身没有迭代器。它们生成外部辅助对象（例如，Java 中基于 Iterator 接口的对象）来传送迭代器状态。

Ruby 的内部迭代器并不总是最好的解决方案。当你需要把迭代器本身作为一个对象时（例如，将迭代器传递给一个方法，而该方法需要访问由迭代器返回的每一个值），它的表现就欠佳了。另外，使用 Ruby 內建的迭代器模式也难以实现并行迭代两个收集。

Ruby 1.8 提供了 Generator 库，该库为解决这些问题实现了外部迭代器。

#### 4.2.2 事务 Blocks

block 可以用来定义必须运行在事务控制环境下的代码。比如文件的打开关闭：

```ruby
File.open_and_process("testfile", "r") do |file|
  while line = file.gets
    puts line
  end
end
```

 如果 `File.open` 有个关联的 block，那么该 block 将被调用，且参数是该文件对象，当 block 执行结束时文件会被关闭。这意味着 `File.open` 有两种不同的行为：当和 block 一起调用时，它会执行该 block 并关闭文件；当单独调用时，它会返回文件对象。

#### 4.2.3 Blocks 可以作为闭包

以下使用 block 解决点唱机的相关按钮，避免了子类冗余等问题：

```ruby
songlist = SongList.new

class JukeboxButton < Button
  def initialize(label, &action)
    super(label)
    @action = action
  end
  
  def button_pressed
    @action.call(self)
  end
end

start_button = JukeboxButton.new("Start")	{ songlist.start }
pause_button = JukeboxButton.new("Pause")	{ songlist.pause }
```

关键在于 `JukeboxButton#initialize` 的第二个参数。如果定义方法时在最后一个参数前加一个 `&`，那么当调用该方法时，Ruby 会寻找一个 block。block 将会被转化成 Proc 类的一个对象，并赋值给参数。这样可以像任意其他变量一样处理该参数。

当我们创建 Proc 对象时，得到的不仅仅是一堆代码。和 block（以及 Proc 对象）关联在一起的还有定义 block 时的上下文，即 self 的值、作用域内的方法、变量和常量。即使 block 被定义时的环境早已消失了，block 仍然可以使用其原始作用域中的信息。在其他语言中，这种特性称之为闭包。比如：

```ruby
def n_times(thing)
  return lambda {|n| thing * n}
end

p1 = n_times(23)
p1.call(3)	# ->	69
p1.call(4)	# ->	92
p2 = n_times("Hello ")
p2.call(3)	# ->	"Hello Hello Hello "
```

### 4.3 处处皆是容器

容器、block 和迭代器是 Ruby 的核心概念。
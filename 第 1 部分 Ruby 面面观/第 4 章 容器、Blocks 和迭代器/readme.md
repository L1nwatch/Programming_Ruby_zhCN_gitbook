容器（containers），是指含有一个或多个对象引用的对象

### 4.1 容器

#### 4.1.1 数组

数组由 `[ ]` 操作符来进行索引。和 Ruby 的大多数操作符一样，它实际上是一个方法，因此可以被子类重载。

也可以使用一对数字 `[start, count]` 来访问数组。这将返回一个包含从 start 开始的 count 个对象引用的新数组。

```ruby
a[1, 3]	->	[3, 5, 6]
```

还可以使用 range 来对数组进行索引，其开始和结束位置被两个或 3 个点分割开。两个点的形式包含结束位置，而 3 个点的形式不包含。

```ruby
a[1..3]	->	[3, 5, 6]
```

`[ ]` 操作符有一个相应的 `[ ]=` 操作符，它可以设置数组中的元素。如果下标是单个整数，那么其位置上的元素将被赋值语句右边的东西所替换。造成的任何间隙将由 nil 来填充。

如果 `[ ]=` 的下标是两个数字（起点和长度）或者是 range，那么原数组中的那些元素将被赋值语句右边的东西所替换。如果长度是 0，那么赋值语句右边的东西将被插入到数组的起点位置之前，且不会删除任何元素。如果右边本身是一个数组，那么其元素将替换掉原数组对应位置上的东西。如果索引下表选择的元素个数和赋值语句右边的元素个数不一致，那么数组会自动调整其大小。

```ruby
a = [1, 3, 5, 7, 9]	->	[1, 3, 5, 7, 9]
a[2, 2]	= 'cat'		->	[1, 3, "cat", 9]
a[2, 0] = 'dog'		->	[1, 3, "dog", "cat", 9]
a[1, 1]	= [9, 8, 7]	->	[1, 9, 8, 7, "dog", "cat", 9]
a[0..3]	= []		->	["dog", "cat", 9]
a[5..6] = 99, 98	->	["dog", "cat", 9, nil, nil, 99, 98]
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


一个简单的测试：

```ruby
require "roman"
r = Roman.new(1)
fail "'i' expected" unless r.to_s == "i"
```

Ruby 自带了一个预安装的 `Test::Unit` 框架

### 12.1 `Test::Unit` 框架

`Test::Unit` 框架基本上是将 3 个功能包装到一个整洁的包中。

*   它提供了一种表示单个测试的方式
*   它提供了一个框架来组织测试
*   它提供了灵活的方式来调用测试

#### 12.1.1 断言 == 预期的结果

```ruby
require "roman"
require "test/unit"
class TestRoman < Test::Unit::TestCase
  def test_simple
    assert_equal("i", Roman.new(1).to_s)
    assert_equal("ix", Roman.new(9).to_s)
  end
end
```

测试一些其他的：

```ruby
require "roman"
require "test/unit"
class TestRoman < Test::Unit::TestCase
  def test_range
    assert_raise(RuntimeError) { Roman.new(0) }
    assert_nothing_raised() { Roman.new(1) }
    assert_nothing_raised() { Roman.new(4999) }
    assert_raise(RuntimeError) { Roman.new(5000) }
  end
end
```

每个断言的最后一个参数是一条消息，会在任何错误消息之前输出。一般它是不需要的，因为 `Test::Unit` 的消息通常已经很适用了。

```ruby
require "test/unit"
class TestWhichFail < Test::Unit::TestCase
  def test_reading
    assert_not_nil(ARGF.read, "Read next line of input")
  end
end
```

### 12.2 组织测试

单元测试，可以很自然地被组织成更高层的形式，叫做测试用例（test case）。

表示测试的类必须是 `Test::Unit::TestCase` 的子类。含有断言的方法名必须以 test 开头。`Test::Unit` 适用反射（reflection）来查找要运行的测试，而只有以 test 开头的方法才符合条件。

在一个 TestCase 类中，setup 的方法将在每个测试方法之前运行，而 teardown 的方法在每个测试方法结束之后运行。

```ruby
require "test/unit"
require "playlist_builder"
require "dbi"

class TestPlaylistBuilder < Test::Unit::TestCase
  def setup
    @db = DBI.connect("DBI:mysql:playlists")
    @pb = PlaylistBuilder.new(@db)
  end
  def teardown
    @db.disconnect
  end
  def test_empty_playlist
    assert_equal([], @pb.playlist())
  end
  # ...
end
```

### 12.3 组织和运行测试


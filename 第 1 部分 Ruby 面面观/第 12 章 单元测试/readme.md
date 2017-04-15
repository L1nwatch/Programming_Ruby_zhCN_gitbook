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

可以使用下面的形式从命令行运行测试：

```shell
ruby test_roman.rb
```

如果希望可以让它运行一个特定的测试方法：

```ruby
ruby test_roman.rb --name test_range
```

#### 12.3.1 何处存放测试

一般的解决方法是建立一个 `test/` 目录，将你所有的测试源文件放置其中。

```shell
roman
	|_	lib/
		|_	roman.rb
		|_	other files...
	|_ test/
		|_ test_roman.rb
		|_ other files...
	|_ other stuff
```

 如何告诉 Ruby 到哪里查找要测试的库文件？一种不太可靠的方法是，将路径放到测试文件的 `require` 语句中，然后从 `test/` 子目录中运行测试。

一种稍好些的解决方法是，从被测试库的父目录中运行测试。

推荐的做法，在你的测试代码的头部中添加下面的代码行：

```ruby
$:.unshift File.join(File.dirname(__FILE__), "..", "lib")
require ...
```

这种技法可以工作，是因为测试代码相对被测试代码的存放位置是已知的。它首先得出测试文件运行所在的目录，然后构造被测试文件的路径。这个目录被加入加载路径（变量 `$:`）的前部。之后，类似如 `require 'roman'` 的代码将首先搜索到被测试的库

#### 12.3.2 测试套件

你可以将测试用例一并组成测试套件（`test suite`），然后以组的形式运行它们。

你所要做的，就是创建一个要求加载 `test/unit` 的 Ruby 文件，然后要求加载每个你希望成组的、包含测试用例的文件。

命名约定：测试用例在以 `tc_xxx` 命名的文件中，而测试套件在 `ts_xxx` 命名的文件中。

```ruby
# file ts_dbaccess.rb
require "test/unit"
require "tc_connect"
require "tc_query"
# ...
```

现在，如果你运行 `ts_dbaccess.rb`，将会执行所要求加载的文件中的全部测试用例。

方法大全：

```ruby
assert
assert_nil
assert_not_nil
assert_equal
assert_not_equal
assert_in_delta
assert_raise
assert_nothing_raised
assert_instance_of
assert_kind_of
assert_respond_to
assert_match
assert_no_match
assert_same
assert_not_same
assert_operator
assert_throws
assert_send
flunk
```
---
layout: post
title: Go 单元测试
date: 2015-11-19 10:35:56 +0800
categories: Go
---

### Go 的单元测试

在用 Go 写基本的如 stack、list等数据结构时候，要测试自己是否写对了，于是就想看看 Go 的单元测试是怎么写的，查看官方文档：

> Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试，`testing`框架和其他语言中的测试框架类似，你可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例.

实际操作就是建立一个 gotest 目录。在该目录下面创建两个文件：gotest.go和gotest_test.go。

1. gotest.go:这个文件里面我们是创建了一个包，里面有一个函数实现了除法运算:

```Go
		package gotest
		
		import (
			"errors"
		)
		// 除法函数
		func Division(a, b float64) (float64, error) {
			if b == 0 {
				return 0, errors.New("除数不能为0")
			}
			return a / b, nil
		}
```

1. gotest_test.go:这是我们的单元测试文件，但是记住下面的这些原则：

   - 文件名必须是`_test.go`结尾的，这样在执行`go test`的时候才会执行到相应的代码
   - 你必须import `testing`这个包
   - 所有的测试用例函数必须是`Test`开头
   - 测试用例会按照源代码中写的顺序依次执行
   - 测试函数`TestXxx()`的参数是`testing.T`，我们可以使用该类型来记录错误或者是测试状态
   - 测试格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以为任意的字母数字的组合，但是首字母不能是小写字母[a-z]，例如`Testintdiv`是错误的函数名。
   - 函数中通过调用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法，说明测试不通过，调用`Log`方法用来记录测试的信息。

   下面是我们的测试用例的代码：

```Go
		package gotest
		
		import (
			"testing"
		)
		// 测试函数一
		func Test_Division_1(t *testing.T) {
			if i, e := Division(6, 2); i != 3 || e != nil { //try a unit test on function
				t.Error("除法函数测试没通过") // 如果不是如预期的那么就报错
			} else {
				t.Log("第一个测试通过了") // 记录一些你期望记录的信息
			}
		}
		// 测试函数二
		func Test_Division_2(t *testing.T) {
			t.Error("就是不通过")
		}
```

看看输出结果。在当前目录下执行 `go test`命令：

![](http://ww1.sinaimg.cn/large/b10d1ea5gw1fbq6nzd1a4j21kw0c9wgt.jpg)

如上结果只能返回失败的测试的信息，对于测试通过的不予显示。如果我们想看所有测试函数的信息，则可以执行 `go test-v`:

![](http://ww1.sinaimg.cn/large/b10d1ea5gw1fbq6oixrxoj21kw0ggq62.jpg)

从上面可以看出执行的测试函数、测试是否通过、测试时间以及测试函数所留下的信息。接着从一段基本的代码看看。下面这段代码是用 Go 来实现 stack 这种基本的数据结构。设计栈要满足以下这些条件或者称为功能：

- 初始化栈
- 返回栈的长度
- 判断堆栈是否为空
- 进出堆栈的功能

看一下具体代码的实现：

```go
package stack

import (
	"errors"
)

// Stack is a FILO data structure.
// Stack supports two major methods: push and pop.
type Stack struct {
	data []interface{}
	len  int
}

// 初始化空堆栈
func New() *Stack {
	return &Stack{}
}

// 返回堆栈的长度
func (s *Stack) Length() int {
	return s.len
}

// 判空函数
func (s *Stack) IsEmpty() bool {
	return s.len == 0
}

// 弹出栈顶元素
func (s *Stack) Peek() interface{} {
	return s.data[s.len-1]
}

// 压进最新的元素进堆栈
func (s *Stack) Push(element interface{}) {
	s.len++
	s.data = append(s.data, element)
}

// 弹出栈顶的元素并且返回该元素
func (s *Stack) Pop() (interface{}, error) {

	if s.IsEmpty() {
		return nil, errors.New("Pop an empty stack.")
	}
  	// 调用 peek() 函数
	element := s.Peek()
    // 长度和堆栈中的内容减少一
	s.data = s.data[:s.len-1]
	s.len--
	return element, nil
}
```

如下为两个测试代码中的测试函数，下面分析一下基本的结构：

```go
// 测试 Push 和 Peek 操作
func TestPushAndPeek(t *testing.T) {
	assert := assert.New(t)
	// 初始化
	s := stack.New()
	// 压入一个元素
	s.Push("a")
  	// 如果堆栈的长度不为 1 的话，则测试不通过
	assert.Equal(s.Length(), 1, "Stack length should be 1 after pushing an element, got %d", s.Length())
	assert.Equal(s.Peek(), "a", "Expect element `a`, got %v", s.Peek())
	// 再次压入一个元素
	s.Push(42)
	assert.Equal(s.Length(), 2, "Stack length should be 2 after pushing another element, got %d", s.Length())
	assert.Equal(s.Peek(), 42, "Expect element `42`, got %v", s.Peek())
}
```

测试判空函数：

```go
func TestIsEmpty(t *testing.T) {
	assert := assert.New(t)

	s := stack.New()
	assert.True(s.IsEmpty(), "New stack should be empty")
	// 压入一个元素，期待的结果应该是 IsEmpty()为 False
	s.Push(42)
	assert.False(s.IsEmpty(), "Stack should not be empty.")
	// 弹出，IsEmpty() 应该为 True
	s.Pop()
	assert.True(s.IsEmpty(), "New stack should be empty")
}
```

执行测试函数之后得到如下结果：

![](http://ww4.sinaimg.cn/large/b10d1ea5gw1fbq6p27pn1j21kw08lmyt.jpg)

### 性能测试

压力测试用来检测函数(方法）的性能，形式和编写单元功能测试的方法类似，但需要注意以下几点：

- 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母

  ```
    func BenchmarkXXX(b *testing.B) { ... }
  ```

- `go test`不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`, 例如`go test -test.bench=".*"`表示测试全部的压力测试函数。

- 在压力测试用例中,请记得在循环体内使用`testing.B.N`,以使测试可以正常的运行。

- 文件名也必须以`_test.go`结尾。

下面以斐波那契函数来进行压力测试：

```go
package lib

//斐波那契数列
//求出第n个数的值
func Fibonacci(n int64) int64 {
	if n < 2 {
		return n
	}
	return Fibonacci(n - 1) + Fibonacci(n - 2)
}
```

测试函数：

```go
package lib

import (
	"testing"
)

//性能测试
func BenchmarkFibonacci(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Fibonacci(10)
	}
}
```

执行测试命令行代码 `go test -test.bench=".*"`得到如下结果：

![](http://ww2.sinaimg.cn/large/b10d1ea5gw1fbr3twdtdkj21kw096wgd.jpg)

其中 3000000 表现该函数执行的次数，418 表示执行一次需要 418 ns。

以上就是 Go 测试的基本内容，谢谢阅读！！！
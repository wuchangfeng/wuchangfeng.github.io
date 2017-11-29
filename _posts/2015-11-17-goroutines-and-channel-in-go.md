---
layout: post
title: Go 中的 Goroutine 和 Channel
date: 2015-11-17 21:28:21 +0800
categories: Go
---

### 运行时异常和 panic

跟许多语言一样，Go 中也有相应的机制，来保证程序的运行正常。当发生像数组下标越界或类型断言失败这样的运行错误时，Go 运行时会触发*运行时 panic*，伴随着程序的崩溃抛出一个 `runtime.Error` 接口类型的值。类似于书上说的，其实跑出异常并不是重点，重要的是能够在程序发生这个错误的时候，不就程序，让其继续执行。

```go
package main

import (
    "fmt"
)

func badCall() {
    panic("bad end")
}

func test() {
   // 注释一
    defer func() {
        if e := recover(); e != nil {
            fmt.Printf("Panicing %s\r\n", e)
        }
    }()
    badCall()
    fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
    fmt.Printf("Calling test\r\n")
    test()
    fmt.Printf("Test completed\r\n")
}
```

像上面这个实例，如果我们只定义了 panic 异常机制，程序最多只会抛出这个执行异常，会中断程序的正常执行。好了，现在我们结合 defer 和 recover 函数，在程序发生异常时候，不仅能够抛出异常，而且同时能够在这个状态中恢复过来，继续执行后续程序。

### 协程的概念

在其他语言中，比如 C#，Lua 或者 Python 都有协程的概念。这个名字表明它和 Go协程有些相似，不过有两点不同：

- Go 协程意味着并行（或者可以以并行的方式部署），协程一般来说不是这样的
- Go 协程通过**通道**来通信，而不是通过共享内存。协程通过让出和恢复操作来通信

Go 协程比一般的协程更强大，也很容易从协程的逻辑复用到 Go 协程。Go 中的协程通过关键字 Go 来调用一个方法或者函数来实现的。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("In main")

	go longWait()
	go shortWait()
	fmt.Println("About to sleep in main()")
	time.Sleep(10 * 1e9)
	fmt.Println("At the end of main()")
}

func longWait() {
	fmt.Println("Beginning longWait()")
	time.Sleep(5 * 1e9) // sleep for 5 seconds
	fmt.Println("End of longWait()")
}

func shortWait() {
	fmt.Println("Beginning shortWait()")
	time.Sleep(2 * 1e9) // sleep for 2 seconds
	fmt.Println("End of shortWait()")
}

```

我们初步来体验一下协程的魅力，如上述程序，输出如下结果，可以看到 Main() 程序在开启两个 Go 程序之后，就继续去**执行自己的逻辑了**，等待两个 Go 程序执行完毕之后，返回结果，最后 Main 程序结束。

>  ``` go
>  In main
>  About to sleep in main()
>  Beginning longWait()
>  Beginning shortWait()
>  End of shortWait()
>  End of longWait()
>  At the end of main()
>  ```

对比的实验，我们去掉 Go 关键字，Main 程序遇到额外的函数，会跟到额外的函数中去执行逻辑，等到额外的程序执行完毕之后，再跳回到 Main 程序中执行：

>  ``` go
>  In main
>  Beginning longWait()
>  End of longWait()
>  Beginning shortWait()
>  End of shortWait()
>  About to sleep in main()
>  At the end of main()
>  ```

### 使用通道进行协程间通信

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)
	go sendData(ch)
	go getData(ch)
	// 注释一
	time.Sleep(1e9)
}

// 往通道中塞进数据
func sendData(ch chan string) {
	ch <- "A"
	ch <- "g"
	ch <- "f"
	ch <- "e"
	ch <- "d"
	ch <- "c"
}

// 提取通道中的数据
func getData(ch chan string) {
	var input string
	//time.Sleep(1e9)
	for {
		input = <-ch
		fmt.Printf("%s", input)
	}
}
```

如上代码中，sendData() 函数通过通道 ch 发送了 5 个字符串。另外，根据书上说的，尝试注释掉注释一处的睡眠时间，会发生什么？很简单，sendData() 根本没时间去发送出去数据，因为在开启 sendData() 协程之后，程序的执行逻辑又立马回到了 Main 函数中，这个时候，如果程序不去睡眠或者等待一会的话，Main 马上就结束，与此同时 sendData() 程序还未获得足够的时间，也要被迫一起死亡了。

### 通道阻塞

默认情况下，通信是同步且无缓冲的：在有接受者接收数据之前，发送不会结束。可以想象一个无缓冲的通道在没有空间来保存数据的时候：必须要一个**接收者**准备好接收通道的数据然后发送者可以直接把数据发送给接收者。所以通道的发送/接收操作在对方准备好之前是**阻塞**的。

```go
package main

import "fmt"

func main() {
    ch1 := make(chan int)
    go pump(ch1)       // pump hangs
    fmt.Println(<-ch1) // prints only 0
}

func pump(ch chan int) {
    for i := 0; ; i++ {
        ch <- i
    }
}
```

这里并没有一个接受者来接受发送者发送的数据，所以通道是阻塞的。故而打印出通道的数值，也只是默认数值 0 而已。这里我猜想的，如果要是验证的话，可以在 通道 ch1 初始化之后验证。

### go 中的死锁

如果发送者和接受者在通信期间互相阻塞对方，就会造成死锁状态。Go 运行时会检查并 panic，停止程序。

```go
package main

import (
	"fmt"
)

func f1(in chan int) {
	fmt.Println(<-in)
}

func main() {
	out := make(chan int)// 0
  	out <- 2// 1
	go f1(out)// 2
}
```

如上并没有缓存的 channel ，在 out <- 2 时候，并没有接受者去读取数据。故而我们的解决办法有两个，将 1 和 2 对调先开启通道，然后在向通达灌输数据(这里看一下上述实例二就明白了)以及给 0 处增加一个缓冲通道：

> out := make(chan int, 1)

### 同步通道-使用带缓冲的通道

通常情况下，无缓冲通道只能包含一个元素，这样就很局限了，我们可以给通道声明一个缓存，使用 make 命令扩展容量：

```go
buf := 100
ch1 := make(chan string, buf)
```

好了，现在我们给通道设置了缓冲功能，就不会像之前那样，轻易的发生阻塞情况了，这一点其实也可以理解：

```go
package main

import "fmt"
import "time"

func main() {
	c := make(chan int)
	go func() {
		time.Sleep(15 * 1e9)
		x := <-c
		fmt.Println("received", x)
	}()
	fmt.Println("sending", 10)
	c <- 10
	fmt.Println("sent", 10)
}
/* Output:
sending 10
(15 s later):
received 10
sent 10
*/
```

如上实例程序，我们先开启一个通道。接着往通道发送数据，由于没有缓冲功能，我们必须得等到子程序即协程执行完毕才可以切回到主程序，所以如下打印结果就能反映出示例这种情况。

在下面这个程序中，有点难理解啊。其实在有缓冲通道情况下，我们将数据放进通道，不会造成任何阻塞的情况，这个时候，我们的程序执行逻辑可以立马切回到 Main 函数中，这样，由于 Main 函数没有睡眠或者延时，导致整个程序会立马顺利结束：

```go
package main

import "fmt"
import "time"

func main() {
	c := make(chan int, 50)
	go func() {
		time.Sleep(15 * 1e9)
		x := <-c
		fmt.Println("received", x)
	}()
	fmt.Println("sending", 10)
	c <- 10
	fmt.Println("sent", 10)
}
/* Output:
sending 10
sent 10   // prints immediately
no further output, because main() then stops
*/
```

如果容量大于 0，通道就是异步的了：缓冲满载（发送）或变空（接收）之前通信不会阻塞，元素会按照发送的顺序被接收。如果容量是 0 或者未设置，通信仅在收发双方准备好的情况下才可以成功。

### 通道的工厂模式

不将通道作为参数传递给协程，而用函数来生成一个通道并返回（工厂角色）；函数内有个匿名函数被协程调用。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    stream := pump()
    go suck(stream)
    time.Sleep(1e9)
}

func pump() chan int {
    ch := make(chan int)
  	// 开启一个匿名函数的携程
    go func() {
        for i := 0; ; i++ {
            ch <- i
        }
    }()
    return ch
}

func suck(ch chan int) {
    for {
        fmt.Println(<-ch)
    }
}
```

### Go 中的信号量模式

```go
func compute(ch chan int){
  	// 计算结束时，向通道放置信号量
    ch <- someComputation() 
}

func main(){
    ch := make(chan int)    
    go compute(ch)      // stat something in a goroutines
    doSomethingElseForAWhile()
  	// 获取通道中的信号量数值
    result := <- ch
}
```

有时候并不关心，通道信号量所返回的通道数值：

```go
ch := make(chan int)
go func(){
    // doSomething
    ch <- 1 // Send a signal; value does not matter
}
doSomethingElseForAWhile()
// 直到 func() 完成，并且这里并不在乎信号量的数值
<- ch   
```

### 用带缓冲通道实现一个信号量

```go
package main

import "fmt"

// integer producer:
func numGen(start, count int, out chan<- int) {
	for i := 0; i < count; i++ {
		out <- start
		start = start + count
	}
	close(out)
}

// integer consumer:
func numEchoRange(in <-chan int, done chan<- bool) {
	for num := range in {
		fmt.Printf("%d\n", num)
	}
	done <- true
}

func main() {
	numChan := make(chan int)
	done := make(chan bool)
	go numGen(0, 10, numChan)
	go numEchoRange(numChan, done)
	<-done
}
```

### 关闭通道

```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan string)
	go sendData(ch)
	getData(ch)
}

func sendData(ch chan string) {
	ch <- "Washington"
	ch <- "Tripoli"
	ch <- "London"
	ch <- "Beijing"
	ch <- "Tokio"
  	// 关闭通到，不能再发送数据
	close(ch)
}

func getData(ch chan string) {
	for {
      	// 获取通道 open 状态
		input, open := <-ch
		if !open {
			break
		}
		fmt.Println("%s", input)
	}
}
```

### select 切换协程

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go pump1(ch1)
	go pump2(ch2)
	go suck(ch1, ch2)
	time.Sleep(1e9)
}

func suck(ch1, ch2 chan int) {
	for {
      	// select 处于监听模式
		select {
        // 1通道有数据的话，取出
		case v := <-ch1:
			fmt.Printf("Received on channel 1: %d\n", v)
        // 2通道有数据的话，取出
		case v := <-ch2:
			fmt.Printf("Received on channel 2: %d\n", v)
		}
	}
}

func pump1(ch chan int) {
	for i := 0; ; i++ {
		ch <- i * 2
	}
}

func pump2(ch chan int) {
	for i := 0; ; i++ {
		ch <- i + 5
	}
}
```

打印出如下实例结果，可以看见数据到底从哪个通道出来具有很大的随机性，其实 select 的作用，也只是看哪一个通道带有数据，就取出来：

```go
[ `go run select.go` | done: 1.172334459s ]
  Received on channel 1: 0
  Received on channel 1: 2
  Received on channel 2: 5
  Received on channel 2: 6
  Received on channel 2: 7
  Received on channel 1: 4
  Received on channel 1: 6
  Received on channel 1: 8
  Received on channel 2: 8
  Received on channel 1: 10
  Received on channel 1: 12
  Received on channel 1: 14
....
```

### 超时处理

超时对于链接外部资源或者其他一些需要花费执行时间的操作程序很重要：

``` go
package main

import (
	"fmt"
	"time"
)

func main() {
	c1 := make(chan string, 1)
	go func() {
		time.Sleep(time.Second * 2)
		c1 <- "result 1"
	}()

	select {
	case res := <-c1:
		fmt.Println(res)
		// 超过 1s 就抛出错误
	case <-time.After(time.Second * 1):
		fmt.Println("timeout 1")
	}

	c2 := make(chan string, 1)
	go func() {
		time.Sleep(time.Second * 2)
		c2 <- "result 2"
	}()
	select {
	case res := <-c2:
		fmt.Println(res)
      // 超过 3s 就抛出错误
	case <-time.After(time.Second * 3):
		fmt.Println("timeout 2")
	}
}
```

### 定时器

顾名思义，定时执行一个任务或者在某段时间间隔内重复执行：

``` go
func main() {

	timer1 := time.NewTicker(time.Second * 2)
	<-timer1.C
	fmt.Println("Timer 1 expired")

	timer2 := time.NewTimer(time.Second)
	go func() {
		<-timer2.C
		fmt.Println("Timer 2 expired")
	}()
	stop2 := timer2.Stop()
	if stop2 {
		fmt.Println("Timer 2 stopped")
	}
}
```

### 打点器

固定时间内重复一个操作，直到我们将其停止：

``` go
func main() {
	ticker := time.NewTicker(time.Millisecond * 500)
	go func() {
		for t := range ticker.C {
			fmt.Println("Tick at", t)
		}
	}()

	time.Sleep(time.Millisecond * 1600)
  	// 停止
	ticker.Stop()
	fmt.Println("Ticker stopped")
}
```


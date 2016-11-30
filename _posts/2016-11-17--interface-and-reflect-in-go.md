---
layout: post
title: Go：Go 中的 Interface 和 Reflect
date: 2016-11-17 20:12:44 +0800
categories: Go
---

### 基本定义

Go 语言不是一种 *传统* 的面向对象编程语言：它里面没有类和继承的概念。而是到处充满了松耦合的类型（结构体或者基本类型）、方法对接口的实现。Go 是唯一结合了**接口值**，静态类型检查（是否该类型实现了某个接口），**运行时动态转换**的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许我们在不改变已有的代码的情况下定义和使用新接口。

但是 Go 语言里有非常灵活的 **接口** 概念，通过它可以实现很多面向对象的特性。如果某个对象实现了某个接口的所有方法，则此对象就实现了此接口

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

- **类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口**。
- **实现某个接口的类型（除了实现接口方法外）可以有其他的方法**。
- **一个类型可以实现多个接口**。
- **接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）**。

当变量被赋值给一个接口类型的变量时，编译器会检查其是否实现了该接口的所有函数，以下实例可以说明这一点：

```go
package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}

// 此处就是结构体Square 实现了接口 Shaper
func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

func main() {
    sq1 := new(Square)
    sq1.side = 5
  	// 注释一
    areaIntf := sq1
    fmt.Printf("The square has area: %f\n", areaIntf.Area())
}
```

对于注释一处，实际上原理代码如下：

```go
var areaIntf Shaper
areaIntf = sq1
// 或者更短一点
areaIntf := Shaper(sq1)
```

现在接口变量(areaIntf)包含一个指向 `Square` 变量的引用，通过它可以调用 `Square` 上的方法 `Area()`.

如果 `Square` 没有实现 `Area()` 方法以及如果 `Shaper` 有另外一个方法 `Perimeter()`，但是`Square` 没有实现它，即使没有人在 `Square` 实例上调用这个方法，编译器也会给出上面同样的错误。编译器将会给出清晰的错误信息：

> ```
> cannot use sq1 (type *Square) as type Shaper in assignment:
> *Square does not implement Shaper (missing Area method)
> ```

### 利用接口实现多态

扩展一下上面的例子，类型 `Rectangle` 也实现了 `Shaper` 接口。接着创建一个 `Shaper` 类型的数组，迭代它的每一个元素并在上面调用 `Area()` 方法，以此来展示多态行为：

```go
package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}
// Square 实现了 Shaper 接口
func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

type Rectangle struct {
    length, width float32
}
// Rectangle 实现了 Shaper 接口
func (r Rectangle) Area() float32 {
    return r.length * r.width
}

func main() {

    r := Rectangle{5, 3} // Area() of Rectangle needs a value
    q := &Square{5}      // Area() of Square needs a pointer
    // shapes := []Shaper{Shaper(r), Shaper(q)}
    // or shorter
    shapes := []Shaper{r, q}
    fmt.Println("Looping through shapes for area ...")
    for n, _ := range shapes {
        fmt.Println("Shape details: ", shapes[n])
        fmt.Println("Area of this shape is: ", shapes[n].Area())
    }
}
```

> ```
> Looping through shapes for area ...
> Shape details:  {5 3}
> Area of this shape is:  15
> Shape details:  &{5}
> Area of this shape is:  25
> ```

在调用 `shapes[n].Area())` 这个时，只知道 `shapes[n]` 是一个 `Shaper` 对象，最后它摇身一变成为了一个 `Square` 或 `Rectangle` 对象，并且表现出了相对应的行为。表现出多态的特征。

### 多重继承

大多数面向对象的语言是不支持多重继承的，因为这样会增加编译器的负担。但是由于 go 中结构体类型的盛行，使得其实现多重继承的功能很简单：组合。

```go
// 类型一
type Phone struct{}

func (p *Phone) Call() string {
	return "Ring Ring"
}

// 类型二
type Camera struct{}

func (c *Camera) TakeAPicture() string {
	return "Click"
}

// 多继承，组合方式
type CameraPhone struct {
	Camera
	Phone
}

func main() {
	cp := new(CameraPhone)
	fmt.Println("Our new CameraPhone exhibits multiple behaviors ...")
	fmt.Println("It exhibits behavior of a Camera: ", cp.TakeAPicture())
	fmt.Println("It works like a Phone too: ", cp.Call())
}
```

输出结果如下：

> 	Our new CameraPhone exhibits multiple behaviors ...
> 	It exhibits behavior of a Camera:  Click
> 	It works like a Phone too:  Ring Ring

### 检测接口类型

个接口类型的变量 `varI` 中可以包含任何类型的值，必须有一种方式来检测它的 **动态** 类型，即运行时在变量中存储的值的实际类型。在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。通常我们可以使用 **类型断言**来测试在某个时刻 `varI` 是否包含类型 `T` 的值：

```go
if _, ok := varI.(T); ok {
    // ...
}
```

如下的实例展示了如何去判断类型变量是否属于某个接口类型以及获得其对应的数值：

```go
package main

import (
    "fmt"
    "math"
)

type Square struct {
    side float32
}

type Circle struct {
    radius float32
}

type Shaper interface {
    Area() float32
}

func main() {
    var areaIntf Shaper
    sq1 := new(Square)
    sq1.side = 5

    areaIntf = sq1
    // Is Square the type of areaIntf?
    if t, ok := areaIntf.(*Square); ok {
        fmt.Printf("The type of areaIntf is: %T\n", t)
    }
    if u, ok := areaIntf.(*Circle); ok {
        fmt.Printf("The type of areaIntf is: %T\n", u)
    } else {
        fmt.Println("areaIntf does not contain a variable of type Circle")
    }
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

func (ci *Circle) Area() float32 {
    return ci.radius * ci.radius * math.Pi
}
```

### 空接口

空接口类似与 Java 或者 C# 中的基类 Object，所有的类型都实现了空接口，其对实现不做任何要求。可以给一个空接口类型的变量 `var val interface {}` 赋任何类型的值。其中 val 可以用许多别名来替代，经常用的就是 Any 或者 any 了。

```go
package main
import "fmt"

var i = 5
var str = "ABC"

type Person struct {
    name string
    age  int
}

type Any interface{}

func main() {
    var val Any
    val = 5
    fmt.Printf("val has the value: %v\n", val)
    val = str
    fmt.Printf("val has the value: %v\n", val)
    pers1 := new(Person)
    pers1.name = "Rob Pike"
    pers1.age = 55
    val = pers1
    fmt.Printf("val has the value: %v\n", val)
    switch t := val.(type) {
    case int:
        fmt.Printf("Type int %T\n", t)
    case string:
        fmt.Printf("Type string %T\n", t)
    case bool:
        fmt.Printf("Type boolean %T\n", t)
    case *Person:
        fmt.Printf("Type pointer to Person %T\n", t)
    default:
        fmt.Printf("Unexpected type %T", t)
    }
}
```

如上我们给空接口类型的变量，赋值，并且尝试了任何类型。结果表明这都是可以的。更进一步，我们在定义二叉树时候，这一点就更为有用了。

### 方法和类型的反射

反射可以在运行时检查类型和变量，例如它的大小、方法和 `动态` 的调用这些方法。这对于没有源代码的包尤其有用。

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
    v := reflect.ValueOf(x)
    fmt.Println("value:", v)
    fmt.Println("type:", v.Type())
  	// 注释一处
    fmt.Println("kind:", v.Kind())
    fmt.Println("value:", v.Float())
    fmt.Println(v.Interface())
    fmt.Printf("value is %5.2e\n", v.Interface())
    y := v.Interface().(float64)
    fmt.Println(y)
}
```

输出结果如下：

> type: float64
> value: 3.4
> type: float64
> kind: float64
> value: 3.4
> 3.4
> value is 3.40e+00
> 3.4

而对于其中的注释一处 v.kind() 其本质的意思是返回 v 最底层的类型，为什么是最底层的类型，看完下面这个例子：

```go
// int 的别名 MyInt
type MyInt int
var m MyInt = 5
v := reflect.ValueOf(m)
```

方法 `v.Kind()` 返回 `reflect.Int`。

### 通过反射去修改类型的值

并不是说有了反射这个黑科技，就能随意的修改数值。因为你有的 value 是不可设置的，判断是否可以设置要用 CanSet() 来判断。

当 `v := reflect.ValueOf(x)` 函数通过传递一个 x 拷贝创建了 v，那么 v 的改变并不能更改原始的 x。要想 v 的更改能作用到 x，那就必须传递 x 的地址 `v = reflect.ValueOf(&x)`。即使 v 现在的类型是 `*float64` 仍然是不可设置的。我们需要间接的借助于 Elem() 函数即 v = v.Elem()，间接的使用指针。

### 反射结构类型

有些时候需要反射一个结构类型。`NumField()` 方法返回结构内的字段数量；通过一个 for 循环用索引取得每个字段的值 `Field(i)`。

同样能够调用**签名在结构上的方法**，使用索引 n 来调用：`Method(n).Call(nil)`。

```go
package main

import (
    "fmt"
    "reflect"
)

type NotknownType struct {
    s1, s2, s3 string
}

func (n NotknownType) String() string {
    return n.s1 + " - " + n.s2 + " - " + n.s3
}

// 定义一个空接口类型变量
var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
    value := reflect.ValueOf(secret) // <main.NotknownType Value>
    typ := reflect.TypeOf(secret)    // main.NotknownType
    // alternative:
    //typ := value.Type()  // main.NotknownType
    fmt.Println(typ)
    knd := value.Kind() // struct
    fmt.Println(knd)

    // 通过反射遍历处结构体中的字段
    for i := 0; i < value.NumField(); i++ {
        fmt.Printf("Field %d: %v\n", i, value.Field(i))
    }

    // call the first method, which is String():
    results := value.Method(0).Call(nil)
    fmt.Println(results) // [Ada - Go - Oberon]
}
```

像 Python，Ruby 这类语言，动态类型是延迟绑定的（在运行时进行）：方法只是用参数和变量简单地调用，然后在运行时才解析。Go 的实现与此相反，通常需要编译器静态检查的支持：当变量被赋值给一个接口类型的变量时，编译器会检查其是否实现了该接口的所有函数。如果方法调用作用于像 `interface{}` 这样的“泛型”上，你可以通过类型断言来检查变量是否实现了相应接口。
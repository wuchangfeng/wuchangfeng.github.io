---
layout: post
title: 为什么你应该全面切换到Kotlin
date: 2018-05-22 20:42:59 +0800
categories: 
---

**[翻译加工来自一片medium的文章](https://medium.com/@magnus.chatt/why-you-should-totally-switch-to-kotlin-c7bbde9e10d5)**，感谢原作者。

### 0. 与Java无缝兼容

Kotlin与Java无缝兼容，意味着你可以逐步从Java迁移到Kotlin，也可以在原有的Java项目中逐步将代码转换成Kotlin.IDEA和AS对Kotlin的系统支持也是非常棒的。

### 1. 熟悉的语法

Kotlin并不是像其他语言一样来自学术界，其来自工业界的大量实践。简单来说，与Java肯定会有一些出入。来看下面的例子吧。

``` kotlin
class Foo {

    val b: String = "b"     // val means 不可修改
    var i: Int = 0          // var means 可修改

    fun hello() {           // 无返回值的函数
        val str = "Hello"
        print("$str World")
    }

    fun sum(x: Int, y: Int): Int {  // 带返回值的函数
        return x + y
    }

    fun maxOf(a: Float, b: Float) = if (a > b) a else b  // 精简写法

}
```

### 2. 字符串插入

相比于Java，Kotlin有着可读性和简洁程度非常好的字符串插入设计：

``` kotlin
val x = 4
val y = 7
print("sum of $x and $y is ${x + y}")  // sum of 4 and 7 is 11
```

### 3. 类型推导

Kotlin有着良好的类型推导机制。

``` kotlin
val a = "abc"                         // 会自动推导是String类型
val b = 4                             // 会自动推导是Int类型

val c: Double = 0.7                   // 显示声明是Double类型
val d: List<String> = ArrayList()     // 显示声明是List类型
```

### 4. 智能转换

判断出特定类型之后，不需要再过渡的去强转。
``` kotlin
if (obj is String) {
    print(obj.toUpperCase())     // obj is now known to be a String
}
```

### 5. 直接判等

现在可以停止去调用equals()函数了，因为==操作符已经具备检查是否具备结构相等的特性了。

``` kotlin
val john1 = Person("John")
val john2 = Person("John")
john1 == john2    // true  结构相等
john1 === john2   // false 引用相等，指的是两者是否同一个对象
```

### 6. 默认参数

再也不需要去定义一些仅仅是参数不同的相似函数了，Kotlin支持了优秀的默认参数机制。

``` kotlin
fun build(title: String, width: Int = 800, height: Int = 600) {
    Frame(title, width, height)
}
```

### 7. 命名参数

当函数具备多个默认参数时候，这个特性也是比较方便的。调用时候，只需覆盖指定要覆盖的参数就行了。

``` kotlin
build("PacMan", 400, 300)                           // equivalent
build(title = "PacMan", width = 400, height = 300)  // equivalent
build(width = 400, height = 300, title = "PacMan")  // equivalent
```

### 8. when表达式

在Kotlin中switch表达式被when表达式替代了。可读性和简洁程度更进一步提升。

``` kotlin
when (x) {
    1 -> print("x is 1")
    2 -> print("x is 2")
    3, 4 -> print("x is 3 or 4")
    in 5..10 -> print("x is 5, 6, 7, 8, 9, or 10")
    else -> print("x is out of range")
}
```

### 9. 属性

能够自定义的去复写Set和Get，在这一点，如果用不到特殊功能，直接用对象去调用属性就好了。

``` kotlin
class Frame {
    var width: Int = 800
    var height: Int = 600

    val pixels: Int
        get() = width * height
}
```

### 10. 数据类

在Kotlin中实体类的定义非常方便，具备Java中的toString、equals、hashCode、copy等方法，但是写法实在是太简单了。

``` kotlin
data class Person(val name: String,
                  var email: String,
                  var age: Int)

val john = Person("John", "john@gmail.com", 112)
```

### 11. 操作符重载

Kotlin中操作符支持重载，这样在可读性方面提升了许多。

``` kotlin
data class Vec(val x: Float, val y: Float) {
    operator fun plus(v: Vec) = Vec(x + v.x, y + v.y)
}

val v = Vec(2f, 3f) + Vec(4f, 1f)
```

### 12. 解析操作

这是一种解析操作，从一个对象或者key-value键值对中解析出特定的字段。
``` kotlin
for ((key, value) in map) {
    print("Key: $key")
    print("Value: $value")
}
```

### 13. Ranges的多样性

这也是属于很多函数式编程语言的特性之一了，Kotlin吸取了这些特性，

``` kotlin
for (i in 1..100) { ... } 
for (i in 0 until 100) { ... }
for (i in 2..10 step 2) { ... } 
for (i in 10 downTo 1) { ... } 
if (x in 1..10) { ... }
```

### 14. 扩展函数

记得以前第一次使用List吗，当你需要Sort()功能时候，到处去Google，最后你发现需要引入Collections.sort(),好了，这个问题解决之后，你需要capitalize字符串，这一次你无法找到已有的工具类，你不得不去写一个StringUtils.

```kotlin
fun String.replaceSpaces(): String {
    return this.replace(' ', '_')
}

val formatted = str.replaceSpaces()
```

当然还有更多的用法如下：

```kotlin
str.removeSuffix(".txt")
str.capitalize()
str.substringAfterLast("/")
str.replaceAfter(":", "classified")
```

### 15. 空指针安全

Kotlin中设定了一系列机制，极力避免Java中令人头疼的空指针问题。

```kotlin
var a: String = "abc"
a = null                // compile error

var b: String? = "xyz"
b = null     
```

如下这种编译就存在不通过，因为b在上述已经声明了可以为空。

```kotlin
val x = b.length        // compile error: b might be null
```

如下这种操作是安全的：

```kotlin
if (b == null) return
val x = b.length        // no problem
```

在Kotlin中可以简化写成如下表达式：

```kotlin
val x = b?.length       // type of x is nullable Int
```

### 16. 更好的lambda表达式

在kotlin中有更好的lambda表达式系统，在可读性和精简之间存在良好的平衡。

```kotlin
val sum = { x: Int, y: Int -> x + y }   // type: (Int, Int) -> Int
val res = sum(4,7)                      // res == 11
```

可以更加简单的写出如下的链式调用，可读性也是非常强的。

```kotlin
persons
    .filter { it.age >= 18 }
    .sortedBy { it.name }
    .map { it.email }
    .forEach { print(it) }
```

以上，感谢原作者。
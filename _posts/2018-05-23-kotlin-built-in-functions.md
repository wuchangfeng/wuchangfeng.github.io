---
layout: post
title: Kotlin 中几种内置函数的用法与区别
date: 2018-05-23 15:19:48 +0800
categories: 
---


记录kotlin中let、apply、run、also、with等函数的用法和区别。

### 0. let 

```kotlin
val a = "hello,kotlin".let{
    println(it)
    3
}

println(a)

hello,kotlin
3
```

定义：

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```
解释：调用 “hello,kotlin”的let函数，it在作用域中替代该对象(hello,kotlin),默认返回函数最后一行

### 1. apply

```kotlin
val a = "hello,kotlin".apply{
    println(this)
}

println(a)    

hello,kotlin
hello,kotlin
```

定义：

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```
解释：函数内可以用this替代调用apply函数的对象，返回值为该对象自己。


### 2. run

``` kotlin
val a = "hello,kotlin".run{
    println(this)
    2
}

println(a)    

hello,kotlin
2
```

定义：

```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}
```
解释：根据上述执行代码，run函数返回闭包内最后一行。

### 3. also

``` kotlin
val a = "hello,kotlin".also{
    println(it)
}

println(a)  

hello,kotlin
hello,kotlin  
```

定义：

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}
```
解释：从源码的定义可以看出，also执行block(闭包)，并返回this，即调用also函数的对象。
 
### 4. with

``` kotlin
val a = with("string") {
    println(this)
    3
}
println(a)

string
3
```

定义：

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```
解释：并不是扩展函数，将指定对象作为函数的参数，在作用域内this替代该对象，返回值为该对象的最后一行。指定的T作为闭包的receiver，使用参数中闭包的返回结果。

以上，注意阅读Kotlin相关高阶函数的源码时候，如果函数中最后一个参数为闭包，那么最后一个参可以不写在括号中，而写在括号后面，如果只有一个参数，括号也可以去掉。
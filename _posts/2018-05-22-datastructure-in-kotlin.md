---
layout: post
title: Kotlin 中 Stack 和 LinkedList 的实现
date: 2018-05-22 20:53:27 +0800
categories: 
---

Kotlin 实现基本的数据结构 Stack 和 LinkedList

### Stack

Java中Stack由List实现，Kotlin中有MutableList，Stack类的基本定义如下,继承Iterator为了迭代遍历：

```kotlin
class Stack<T : Comparable<T>>(list : MutableList<T>) : Iterator<T> 
```

#### 基本属性实现

```kotlin
// stack的count
 var itCounter: Int = 0

// stack内部实现为MutableList
 var items: MutableList<T> = list

// 判断stack是否为null
fun isEmpty(): Boolean = this.items.isEmpty()

// 获取stack的items counte
fun count(): Int = this.items.count()

// tostring操作
 override fun toString(): String {
     return this.items.toString()
 }
```

#### 基本操作实现

```kotlin
// pop操作，弹出栈顶元素即链表最末端元素，可为null
 fun pop(): T? {
    if (this.isEmpty()) {
        return null
    } else {
        val item = this.items.count() - 1
        return this.items.removeAt(item)
    }
}

// 只读操作，不弹出
fun peek(): T? {
    if (isEmpty()) {
        return null
    } else {
        return this.items[this.items.count() - 1]
    }


}

// hasNext操作
override fun hasNext(): Boolean {
    val hasNext = itCounter < count()
    if (!hasNext) itCounter = 0
    return hasNext
}

// 取next元素
override fun next(): T {
    if (hasNext()){
        val topPos : Int = (count() - 1) - itCounter
        itCounter++
        return this.items[topPos]
    }else{
        throw NoSuchElementException("No such element") // 异常不用new哦
    }
}
```

### LinkedList

LinkedList的实现需要Node，然后实现first、last、count以及append等操作。

#### Node 定义

```kotlin
class Node<T>(value : T){
    var value : T = value // value可以是任意类型
    var next : Node<T>? = null // next可以为null
    var previous : Node<T>? = null // pre也可以为null
}
```

#### 基本操作一

```kotlin

// 头结点，引导性作用
var head : Node<T>?= null

// 取决于head是否为null
var isEmpty : Boolean = head == null

// 获取first
fun first() : Node<T>? = head

// 获取last结点，需要一直next才能到达last结点
fun last() : Node<T>?{
    var node = head
    if (node != null){ 
        while (node?.next != null){
            node = node?.next
        }
        return node
    }else{
        return null
    }
}
```

#### 基本操作二

```kotlin
// 获取count，同样通过next计算
fun count():Int {
    var node = head
    if (node != null){
        var counter = 1
        while (node?.next != null){
            node = node?.next
            counter += 1
        }
        return counter
    } else {
        return 0
    }
}

// append操作,在last结点上append
fun append(value : T){
    var newNode = Node(value)
    // 获取当前节点的最后一个节点
    var lastNode = this.last()
    if (lastNode != null){
        newNode.previous = lastNode
        lastNode.next = newNode
    }else{
        head = newNode
    }
}

// 删除操作
fun removeNode(node : Node<T>) : T{
    val prev = node.previous
    val next = node.next

    if (prev != null){
        prev.next = next
    }else{
        head = next
    }
    next?.previous = prev

    node.previous = null // 将断开的节点前后置null
    node.next = null

    return node.value // 返回删除节点的value
}
```

以上，用kotlin实现基本的数据结构stack和linkedlist.
---
layout: post
title: Go 中 Queue 的实现方式
date: 2015-11-21 10:48:32 +0800
categories: 
---

### 需求

队列的特性较为单一，基本操作即初始化、获取大小、添加元素、移除元素等。最重要的特性就是满足先进先出。

### 实现

接下来还是按照以前的套路，一步一步来分析如何利用Go的语法特性实现Queue这种数据结构。

#### 定义

首先定义每个节点Node结构体，照例Value的值类型可以是任意类型，节点的前后指针域指针类型为node

```go
type node struct {
	value interface{}
	prev *node
	next *node
}
```

继续定义链表结构，定义出头结点和尾节点的指针，同时定义队列大小size:

```go
type LinkedQueue struct {
	head *node
	tail *node
	size int
}
```

#### 大小

获取队列大小，只需要获取LinkedQueue中的size大小即可：

```go
func (queue *LinkedQueue) Size() int {
	return queue.size
}
```

#### Peek

Peek操作只需要获取队列队头的元素即可，不用删除。返回类型是任意类型，用接口实现即可。另外如果head指针域为nil，则需要用`panic`抛出异常，一切ok的话，返回队头节点的数值即可：

```go
func (queue *LinkedQueue) Peek() interface{} {
	if queue.head == nil {
		panic("Empty queue.")
	}
	return queue.head.value
}
```

#### 添加

添加操作在队列中是比较重要的操作，也要区分队尾节点是否为nil，根据是否为nil，执行不同的连接操作，最后队列的size要加1，为了不浪费内存新增节点的指针变量要置nil：

```go
func (queue *LinkedQueue) Add(value interface{}) {
	new_node := &node{value, queue.tail, nil}
	if queue.tail == nil {
		queue.head = new_node
		queue.tail = new_node
	} else {
		queue.tail.next = new_node
		queue.tail = new_node
	}
	queue.size++
	new_node = nil
}
```

#### 移除

队列的删除操作也是很简单，无非是节点的断开操作。在此之前，需要判断链表的状态即是否为nil？而后移除的队列最前端的节点，先用一个新的变量节点保存队列前面的节点，进行一系列操作之后，至nil，并将长度减少即可。

```go
func (queue *LinkedQueue) Remove() {
	if queue.head == nil {
		panic("Empty queue.")
	}
	first_node := queue.head
	queue.head = first_node.next
	first_node.next = nil
	first_node.value = nil
	queue.size--
	first_node = nil
}
```

Ok，以上就是用Go的基本语法特性实现Queue的过程。谢谢阅读！！！
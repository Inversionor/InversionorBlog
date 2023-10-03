---
layout: post
title: Golang channel
subtitle: 对于Golang channel的总结笔记
date: 2023-04-29 15:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## channel简介
channel是goroutine之间的通信机制。每个channel都有一个特殊的类型，指明channel可发送数据的类型，如chan int。

## 基本操作
```golang

// 使用make创建一个channel
ch := make(chan int)

// channel通信
// 往channel中写数据
ch <- x
// 从channel中取数据
x := <-ch
<-ch

// 关闭channel
close(ch)

// 单向channel
// 只用于发送in的channel类型
chan<- int
// 只用于接收int的channel类型
<-chan int
```
- 与map类似，channel也对应一个make创建的底层数据结构的引用  
- 当我们复制一个channel或者将channel用于函数参数传递时，我们只是拷贝了一个channel引用，而它的指向将引用同一个channel对象  
- channel的零值是nil  
- 两个相同类型的channel可以使用==比较，如果两个channel引用的是相同的对象，那么比较的结果为真。一个channel也可以和nil进行比较。
- channel本质上也是一个指针

## channel关闭
**关闭channel之后，随后对基于该channel的任何发送操作都将导致panic异常**。  

**而对一个已经关闭了的channel进行接收操作依然可以接收到之前已经成功发送的数据；如果channel中已经没有数据的话将产生一个零值的数据。**

**注：**  
一个goroutine读取一个带缓存的channel时，如果这个channel已经被关闭，则读取完channel中的数据后不会被阻塞，而是一直读到零值；如果这个channel未被关闭，则读取完channel中的数据后goroutine被阻塞。

如何判断一个channel已经被关闭还是真的数据就是零值？
```golang
// 其中c5是一个channel，ok的值为ture表示成功从channels接收到值，false表示channels已经被关闭并且里面没有值可接收。
ch, ok := <-c5
if !ok {
	fmt.Println("channel已经被关闭")
	return
}

// 而使用range循环可以更简洁地遍历channel中的数据
// 它依次从channel接收数据，当channel被关闭并且没有值可接收时跳出循环。
// 其中c5是一个channel
for ch := range c5 {
	fmt.Println("ch:", ch)
}
```

**注：**  
- channel不需要程序员去手动关闭，**不管一个channel是否被关闭，当它没有被引用时将会被Go语言的垃圾自动回收器回收**。  
- **只有当需要告诉接收者goroutine，所有数据已经全部发送时才需要关闭channel**。    
- **试图重复关闭一个channel将导致panic异常，试图关闭一个nil值的channel也将导致panic异常**。

## 无缓存channel与带缓存channel
无缓存channel的发送方要等待接收方取走数据，否则就一直阻塞；带缓存channel，发送方在channel满的情况下想要发数据会阻塞，接收方在channel空的情况下取数据会阻塞。  

无缓存channel更强地保证了每个发送操作与相应的同步接收操作；而对于带缓存的channel，这些操作是解耦的。

## channel底层数据结构
先看看chan.go源码  
```golang
// src/runtime/chan.go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
type waitq struct {
	first *sudog
	last  *sudog
}


func makechan(t *chantype, size int) *hchan {
	... ...
}
```
**hchan数据结构**  

![hchan数据结构](https://s1.ax1x.com/2023/04/30/p93t1sO.jpg)
hchan维护一个**循环队列**，qcount存储队列中现有几个元素；dataqsiz存储循环队列的大小，也就是缓冲区的大小；buf是一个指针，指向该队列；sendx存储发送索引，之后发送的数据将存在这里；recvx存储接收索引，之后从中取数据就从这里取。  

另外还有recvq和sendq用来存储接收goroutine以及发送goroutine的等待队列。  **注：**waitq是sudog的一个双向链表，而sudog实际上是对goroutine的一个封装。    

lock用来保证channel写入和读取数据时线程安全。    

用于创建channel的makechan返回一个hchan类型的指针，所以说channel也是一个指针，特性类似slice & map。  

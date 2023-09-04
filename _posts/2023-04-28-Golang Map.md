---
layout: post
title: Golang Map数据结构
subtitle: 对于Golang Map数据结构的总结笔记
date: 2023-04-28 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## Map简介
Go中的map是一个哈希表的引用，它是一个无序的key/value集合，key不可以重复，通过key可以在常数时间复杂度内检索、更新或者删除对应value。**不建议将浮点数用作key类型**

## 常用操作
```golang
// 通过内置的make函数创建一个map
args := make(map[string]int)

// 通过map字面值的语法创建map，同时初始化一些值
args := map[string]int{
	"id":   1,
	"age": 23,
}
// 创建空的map
args := map[string]int{}

// 通过var声明一个map，它的值为零值nil
var args [int]int{}

// 添加、访问以及修改元素都可以通过下标
args["id"] = 3
args["number"] = 20

// 删除元素使用内置的delete，通过key值来删除
delete(args, "id")

// 如果查找失败将返回value类型对应的零值

// 通过range遍历map中的全部元素

// 某个value的值真的是零值，如何判断他是真的在map中
if age, ok := args["id"]; !ok {说明不在}
// 其中ok是一个布尔值，用来报告元素是否真的存在

```
**注：**  
- map中的元素并不是一个变量，因此不可以对map的元素进行取址操作。原因是map可能随着元素数量增长而重新分配更大的空间，从而导致之前的地址无效。  
- map类型的零值是nil，也就是没有引用任何哈希表，向一个nil值的map存入一个元素将导致一个panic异常  
- var args [int]int{}创建的map值是nil值，而args := map[string]int{}不是  
- map之间不能进行相等比较，要判断两个map是否包含相同的key和value，必须通过一个循环实现。   
- 在Go中，任何创建map的代码最终调用的都是 **runtime.makemap**函数。**makemap**函数返回的是一个*hmap指针，因此创建得到的map是一个指针。

## 底层数据结构
map的底层是一个哈希表，数据通过哈希函数均匀的分布在各个bucket桶中。  
### 哈希值
哈希函数将传入的key进行哈希运算，得到一个唯一的hash值。  
hash值分为**高位哈希值**和**低位哈希值**：
高位哈希值：hash值的高8位，用来确定当前的bucket中有没有所存储的数据  
低位哈希值： hash值的低B位，用来确定当前的数据存在了哪个bucket  **注：B位是map数据结构中用来存储桶的编号的位数**

低位哈希值确定位于哪个桶；高位哈希值用来加快索引，不用完整比较key就能过滤掉不符合的key，加快查询速度；如果确定了key的位置，再比较完整hash值，如果匹配则获取对应的value。

### 解决冲突
使用拉链法来解决冲突

![Go哈希表](https://s1.ax1x.com/2023/04/28/p9lEZQK.png)

## sync.Map
Go语言原生map并不是线程安全的，对它进行并发读写操作的时候，需要加锁。因此Go引入了**sync.map**，它是一种**并发安全**的map。对sync.map的读写不需要加锁。并且它通过空间换时间的方式，使用read和dirty两个map来进行读写分离，降低锁时间来提高效率。
对于以下场景：  
- 当一个key只被写入一次但被多次读取时  
- 当多个goroutines读取、写入和覆盖不相干的key时

这两种情况与Go map搭配单独的Mutex或RWMutex相比，使用sync.Map类型可以大大减少锁的争夺。

**sync.map适用于读多写少的场景**

### sync.map的底层数据结构
```golang
type Map struct {
	mu Mutex
	read atomic.Value // readOnly
	dirty map[interface{}]*entry
	misses int
}
 
// Map.read 属性实际存储的是 readOnly。
type readOnly struct {
	m       map[interface{}]*entry
	amended bool
}
```
在read和dirty中，都涉及到的结构体：
```
type entry struct {
	p unsafe.Pointer // *interface{}
}

```
read和dirty各自维护一套key，key指向的都是同一个value。

![sync.map结构图](https://s1.ax1x.com/2023/04/28/p9lm96x.png)
引自[sync.Map详解](https://blog.csdn.net/weixin_41335923/article/details/124061082)

sync.map的两个map，当从sync.map类型中读数据时，首先会查看read中是否包含所需的元素：  
- 若有，则通过 atomic 原子操作读取数据并返回。  
- 若无，则会判断 read.readOnly 中的 amended 属性，他会告诉程序 dirty 是否包含 read.readOnly.m 中没有的数据；因此若存在，也就是 amended 为 true，将会进一步到 dirty 中查找数据。

sync.map的读操作性能很高。
---
layout: post
title: Golang Context包
subtitle: 对于Golang Context包的总结笔记
date: 2023-10-03 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

本文是我的context学习记录，由于还没怎么使用过context，所以不甚详细，后续会逐渐补充细节。  

## context包概述
context包定义了Context类型，它在API边界和进程之间携带截止时间、取消信号和其他的请求范围的值。  

对于服务器的进入请求应该创建一个Context，向服务器发出的调用应该接收一个Context。他们之间的函数调用链必须传播Context，可以选择将它们替换为使用WithCancel、WithDeadline、WithTimeout或者WithValue创建的派生Context。当一个Context被撤销时，所有由它派生的Context都会被撤销。  

WithCancel、WithDeadline和WithTimeout函数接受一个Context(the parent)并返回一个派生Context(the child)和一个CancelFunc。调用CancelFunc会撤销child及其children，移除parent对child的引用，并停止所有相关的计时器。调用CancelFunc失败会泄露child和它的children，直到parent被撤销或者计时器触发。  

WithCancelCause函数返回一个CancelCauseFunc，接收一个error并且将它记录为撤销原因。在被删除的context或者它的任何一个children上调用Cause检索原因。如果没有指定原因，Cause(ctx)与ctx.Err()返回同样的值。  

## Context定义
Context是一个接口：
```go
type Context interface{
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key any) any
}
```
接口中提供四个抽象的方法，分别是：
1. Deadline()返回这个context的截止时间，如果没有设定，返回ok为false。
2. Done()返回一个channel，context被撤销后，返回的channel就会被关闭，如果这个context永远不会被关闭，则返回nil。Done()是被提供用来在select中使用的，其示例如下：
Done()返回的是一个单向channel，只能取数据，不能存数据，因此取数据会被阻塞，转而执行out <- v，当ctx被撤销，Done()返回的channel被关闭时，<-ctx.Done()可以取到channel中数据类型的零值，从而触发return ctx.Err()，Stream函数结束。
```go
func Stream(ctx context.Context, out chan<- Value) error {
	for {
		v, err := DoSomething(ctx)
		if err != nil {
			return err
		}
		select {
		case <-ctx.Done():
			return ctx.Err()
		case out <- v:
		}
	}
}
```
3. Err()当Done()没有被关闭，返回空；如果Done()被关闭，返回非空。
4. Value()返回与这个context相关联的对应于key的值，如果没有值与key对应，则返回nil.


## Context应用场景
1. 控制子goroutine的生命周期(常见)
2. 上下文信息传递，比如处理http请求，在请求处理链路上传递信息
3. 超时控制的方法调用
4. 可以取消的方法调用

## 注意事项
使用Context的程序应该遵循以下规则，以保证包之间的接口一致，并使静态分析工具来检查上下文传播：  
1. 不要将Contexts存储在struct类型中；取而代之的是，将Context显式地传递给每个需要它的函数。Context应该是第一个参数，通常命名为ctx，代码示例如下：  
```go
func DoSomeThing(ctx context.Context, arg Arg){
	// ... use ctx ...
}
```
2. 不要传递一个空的Context(nil)，即使函数允许这样做。传递Context.TODO如果你不确定该使用哪个Context。
3. 仅将context Values用于传输进程和API的请求作用域数据，不要用于向函数传递可选参数。
4. 同一个Context可以被传递给运行在不同goroutine中的函数；多个goroutine同时使用Contexts是安全的。




## 参考文献
[1] Go. context[EB/OL]. [2023-10-03]. https://pkg.go.dev/context@go1.21.1.  
[2] AllenWu. Golang Context 详细原理和使用技巧[EB/OL]. [2023-10-03]. https://zhuanlan.zhihu.com/p/587486673?utm_id=0.  
[3] zinx. zinx-issue#248[EB/OL]. [2023-10-03]. https://github.com/aceld/zinx/issues/248.
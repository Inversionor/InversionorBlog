---
layout: post
title: Golang Panic异常
subtitle: 对于Golang Panic异常的总结笔记
date: 2023-04-29 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## Panic简介
**数组越界访问、空指针引用**等**运行时错误**会引起**panic异常**。  
当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟的函数（defer机制）。随后，程序崩溃并输出日志信息。日志信息包括panic value和函数调用的堆栈跟踪信息。panic value通常是某种错误信息。对于每个goroutine，日志信息中都会有与之相对的，发生panic时的函数调用堆栈跟踪信息。

## 内置的panic函数
不是所有的panic异常都来自运行时，**直接调用内置的panic函数**也会引发panic异常；**panic函数接受任何值作为参数**。当某些不应该发生的场景发生时，我们就应该调用panic。
```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Printf("main开始")
	s := []int{}
	go checkSlice(s)
	time.Sleep(time.Second * 10)
	fmt.Printf("main结束")
}

func checkSlice(s []int) {
	time.Sleep(time.Second * 3)
	i := 0
	for i < 200 {
		fmt.Println(i)
		if i == 50 {
			panic(i)
		}
		i++
	}
	defer func() {
		fmt.Println("checkSlice中的defer被执行")
	}()
}
```
**输出：**
```shell
...
47
48
49
50
panic: 50

goroutine 6 [running]:
main.checkSlice({0x0?, 0x0?, 0x0?})
        D:/golang/gotest/panic.go:22 +0xd4
created by main.main
        D:/golang/gotest/panic.go:11 +0x6a
exit status 2

```

## 注意事项
**由于panic会引起程序的崩溃，因此panic一般用于严重错误。**
**对于大部分漏洞，我们应该使用Go提供的错误机制，而不是panic，尽量避免程序崩溃。**  

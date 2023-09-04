---
layout: post
title: Golang string
subtitle: 对于Golang string的总结笔记
date: 2023-04-26 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## string字符串
摘自Golang源码对于string的注释：
>string is the set of all strings of 8-bit bytes, conventionally but not necessarily representing UTF-8-encoded text. A string may be empty, but not nil. Values of string type are immutable.

- string是8比特字节的集合  
- 通常但不一定是UTF-8编码的文本  
- string可以为空，但不会是nil  
- 不可以被修改

## 常用操作
```
//内置的len函数可以返回一个字符串中的字节数目
str := "..."
fmt.Println(len(str))
//也可以通过索引读取第i个字节的字节值
fmt.Println(str[1])
//支持切片操作，左闭右开取子串
fmt.Println(str[1:4])
//切片操作可以省略其中一个或两个参数，如果省略左边，则默认为0，如果省略右边，则默认为len(str)
fmt.Println(str[:4])
fmt.Println(str[4:])
//两个都省略则取整个字符串
fmt.Println(str[:])
//支持+运算符，将两个字符串连接构造一个新字符串
fmt.Println("hello"+str)
```
## 字符编码
Go的字符串采用UTF8编码，对于一些编码超过一个字节的字符（比如汉字），如果按照索引来遍历字符串，由于索引对应的是一个一个字节，因此，会不能读取出来单个汉字，只会读出汉字对应的那三个字节的字节码。  
但是可以用range循环来处理字符串，这时会自动隐式解码UTF8字符串。
```
str := "hello ,你好！  world"
//使用下标索引遍历字符串
for i := 0; i<len(str); i++ {
	fmt.Printf("i:%d char:%q char:%d \n",i,str[i],str[i])
}
//使用range来遍历字符串
for i, j := range str {
	fmt.Printf("i:%d j:%q j:%d \n",i,j,j)
}
```
两种方式得到的输出分别为：  
![index方式](https://s1.ax1x.com/2023/04/26/p9K2fUO.jpg)
![range方式](https://s1.ax1x.com/2023/04/26/p9K2WVK.jpg)

## string底层实现
```
// src/runtime/string.go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```
- str是一个指针，指向字符串的地址  
- len表示字符串长度，也就是字符串所占字节数，但并不是字符串的字符数。如上一节**字符编码**所示，汉字会占用多个字节。  


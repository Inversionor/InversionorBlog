---
layout: post
title: Golang slice数据结构
subtitle: 对于Golang slice数据结构的总结笔记
date: 2023-04-27 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## slice简介
Slice（切片）代表变长的序列，序列中每个元素具有相同的数据类型。slice的底层引用一个数组对象。一个slice包括指针、长度和容量。指针指向第一个slice元素对应的底层数据元素的地址，长度为slice中元素的数目，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数可以获取到slice的长度和容量。

**多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠。**

## 常用操作
```
// 使用make来创建一个slice，其中cap可以省略
// 如省略，默认与len相等
make([]T, len)
make([]T, len, cap)

// 不使用make创建一个slice
// 这样创建的slice的len和cap均为0
copySlice := []int{}
var copySlice []int

// slice的切片操作
// 如果前一个参数省略则默认为0
// 如果后一个参数省略则默认为len(slice1)
slice2 := slice1[2:6]
// 注：切片操作返回的slice2仍与slice1共享相同的底层结构
// 因此如果修改一个切片内的元素值，另一个也会发生改变
// slice2的长度为其实际长度
// slice2的cap为其创造切片时前一个参数到slice1的cap的长度
// 切片操作的两个参数可以超越len(slice1)，但不可超过cap(slice1)

// 向切片追加元素
slice1 = append(slice1, num)
// 追加元素在原len(slice1)之后追加，之后len(slice1)也会发生改变
// 如果追加长度超过了cap，则cap会扩容成为2倍，len为当前slice实际长度

//清空slice可直接置其值为nil
slice1 = nil
```
**注：**
- 只有len范围内的元素是可以直接下标索引到的  
- 长度超越当前cap后，slice会扩容称为当前cap的2倍  
- 切片操作得到的多个slice共享底层的数组数据结构  
- slice的make函数返回一个指针，因此一个slice变量本质是一个指针，指向底层数组中的一些元素的集合。  
  

## append函数
**append函数用来向一个slice追加元素**  
- 首先检查slice底层数组是否有足够容量来保存新添加的元素。如果有足够空间的话，直接在原有底层数组之上扩展，因此输入的slice和返回的slice共享底层数组   
- 如果没有足够的空间来保存新添加的元素，append会扩展cap到当前的2倍，新开一块空间，先把原slice复制到新的空间，再把需要添加的元素复制过去，返回新的空间的slice。  
- 对于通过切片操作产生的两个slice的append，在没有涉及扩展空间之前，他们使用的是同一个底层数组，因此两个slice的append会相互影响，他们append的值可能会相互覆盖；但是在涉及到扩容之后，因为使用了新开辟的空间，他们之间就没有关系了。  ‘

## copy函数
**copy函数会将一个slice的值复制给另一个slice**
```
newSlice := new([]int, len, cap)
copy(newSlice, slice1)
```
- 用于接收的newSlice的len有多大，就复制多少个值过去，按照下标对应复制  
- 参与复制的两个slice不共用空间，相互之间的操作无影响 

## slice与数组的区别
- slice可变长，数组是定长的  
- slice的底层也是数组，但是slice有len和cap两个属性  
- 数组的传递是值传递，而slice作为引用类型，是引用传递，效率更高  
- 多个slice的底层可以是同一个底层数组

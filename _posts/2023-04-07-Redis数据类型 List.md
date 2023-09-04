---
layout: post
title: Redis数据类型 List
subtitle: Redis学习笔记之List
date: 2023-04-07 15:50:00 +0800
categories: Redis
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPr9a7D.jpg'
cover_author: 'Mahdi Yusuf'
cover_author_link: 'https://architecturenotes.co/author/myusuf3/'
tags: 
- Redis  
---

## List简介
List的特征是**单键多值**。Redis列表是简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部（左边）或者尾部（右边）。List的底层是一个双向链表，对两端的操作性能很高，通过索引下标来操作中间的节点性能会较差。

## 常用命令
```
#从(链表)左边/右边插入一个或多个值
lpush/rpush <key><value1><value2><value3>
#按照索引下标获得元素(从左到右)
lrange <key><start><stop>
#从左边/右边取出一个值。值在键在，值光键亡。
lpop/rpop <key>
#从<key1>列表右边取一个值，插入到<key2>列表左边
rpoplpush <key1><key2>
#按照索引下标获得元素(从左到右)
lindex <key><index>
#获取列表长度
llen <key>
#在value前面/后面插入newvalue
linsert <key> before/after <value><newvalue>
#从左边删除n个value，查找value，删n个停止
lrem <key><n><value>
#将列表key下标为index的值替换成value
lset <key><index><value>
```

## List的底层数据结构
List的数据结构为快速链表(quickList)。在列表元素比较少的情况下会使用一块连续的内存存储，这个结构是zipList，也就是压缩链表。当数据量比较多的时候才会改成quickList。quickList的节点是一个一个的zipList。因为传统链表存放指针耗费大量空间所以采用这种方式减小空间冗余，同时也能满足快速的插入删除性能。
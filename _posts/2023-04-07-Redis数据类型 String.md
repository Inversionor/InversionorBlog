---
layout: post
title: Redis数据类型 String
subtitle: Redis学习笔记之String
date: 2023-04-07 14:50:00 +0800
categories: Redis
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPr9a7D.jpg'
cover_author: 'Mahdi Yusuf'
cover_author_link: 'https://architecturenotes.co/author/myusuf3/'
tags: 
- Redis  
---

## 1.Redis String简介
String是Redis最基本的数据类型；它是二进制安全的，意味着它可以包含任何数据；一个Redis字符串value值最多可以是512M。  
## 2.常用命令
```
#給数据库中添加键值对,添加相同的key但value不同，会覆盖之前的
set <key><value>
#查询对应键值
get <key>
#value后追加，将給定的value追加到原值的末尾
append <key><value>
#获得值的长度
strlen <key>
#只有key不存在时，设置key的值
setnx <key><value>
#将key中存储的数字值增1或者减1  注：只能对数字值操作
incr <key>
decr <key>
#设置每次加减的值，而不是只加1减1
incrby/decrby <key><步长>
#同时设置一个或多个的key-value对
mset <key1><value1> <key2><value2>
#同时获取一个或多个value
mget <key1><key2><key3>
#同时设置一个或多个key-value对，当且仅当所有给定key都不存在
msetnx <key1><value1> <key2><value2>
#获得值的范围，前包，后包，相当于截取字符串
getrange <key><起始位置><结束位置>
#用<value>覆写<key>所储存的字符串值，从<起始位置>开始，注：索引从0开始
setrange <key><起始位置><value>
#设置键值的同时，设置过期时间，单位秒
setex <key><过期时间><value>
#以新换旧，设置了新值同时获得旧值
getset <key><value>
```
## String的底层数据结构
String的数据结构为简单动态字符串(Simple Dynamic String，SDS)。采用预分配冗余空间的方式来减少内存的频繁分配。当前字符串实际分配的空间capacity一般要高于实际字符串长度len，当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过了1M，扩容时一次只会多扩1M的空间，字符串的最大长度为512M。
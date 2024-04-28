---
layout: post
title: Redis数据类型 Hash哈希
subtitle: Redis学习笔记之Hash
date: 2023-04-08 10:50:00 +0800
categories: Redis
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPr9a7D.jpg'
cover_author: 'Mahdi Yusuf'
cover_author_link: 'https://architecturenotes.co/author/myusuf3/'
tags: 
- Redis  
---

## Hash哈希简介
Redis hash是一个键值对集合。是一个String类型的field和value的映射表，hash特别适合用于存储对象。即value中存储多个field-value对。找值的话通过key加上field来查找。

## 常用命令
```
#给<key>集合中的 <field>键赋值<value>
hset <key><field><value>
#从<key1>集合<field>取出value
hget <key1><field>
#批量设置hash的值
hmset <key1><field1><value1><field2><value2>
#查看哈希表key中，给定域field是否存在
hexists <key1><field>
#列出该hash集合的所有field
hkeys <key>
#列出该hash集合的所有value
hvals <key>
#为哈希表key中的域field的值加上增量 1 -1
hincrby <key><field><increment>
#将哈希表key中的域field的值设置为value，当且仅当域field不存在
hsetnx <key><field><value>
```
## Hash底层数据结构
Hash类型对应的数据结构有两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。
---
layout: post
title: Redis数据类型 Zset
subtitle: Redis学习笔记之Zset
date: 2023-04-08 19:50:00 +0800
categories: Redis
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPr9a7D.jpg'
cover_author: 'Mahdi Yusuf'
cover_author_link: 'https://architecturenotes.co/author/myusuf3/'
tags: 
- Redis  
---

## Zset简介
Redis Zset有序集合与普通集合set非常相似，是一个没有重复元素的字符串集合。不同之处在于有序集合的每个成员都关联了一个评分（score），这个评分被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以重复。  
因为元素是有序的，所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。访问有序集合的中间元素也是非常快的，因此可以使用有序集合作为一个没有重复成员的智能列表。

## 常用命令
```
#将一个或多个member元素及其score值加入到有序集key当中。
zadd <key><score1><value1><score2><value2>
#返回有序集key中，下标在<start><stop>之间的元素，withscores控制是否显示其分数
zrange <key><start><stop> [WITHSCORES]
#返回有序集key中，所有score值介于min和max之间（包括等于min和max）的成员。有序集成员按score值递增次序排列。
zrangebyscore key minmax [withscores] [limit offset count]
#将上一个命令的结果改为从大到小排列
zrevrangebyscore key maxmin [withscores] [limit offset count]
#为元素的score加上增量
zincrby <key><increment><value>
#删除该集合下，指定值的元素
zrem <key><value>
#统计该集合，分数区间内的元素个数
zcount <key><min><max>
#返回该值在集合中的排名，从0开始
zrank <key><value>
```
## Zset有序集合底层数据结构
Zset底层使用了两个数据结构hash&跳跃表。hash的作用是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值；跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

## 跳跃表（跳表）
跳跃表的效率堪比红黑树，实现远比红黑树简单。从第2层开始找，如果要找的数比当前节点大，就往右边找，如果本层没找着就从当前节点去下一层，在这一层重复上面的操作，以此类推......
![跳表结构](https://github.com/lim-yoona/lim-yoona.github.io/blob/master/images/%E8%B7%B3%E8%A1%A8%E5%9B%BE%E8%A7%A3.jpg?raw=true)  
弹幕：为什么不采用b树是因为Redis在内存中，为什么不采用平衡二叉树，是因为要范围查找。

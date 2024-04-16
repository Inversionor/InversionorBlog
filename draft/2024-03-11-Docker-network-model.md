---
layout: post
title: 'Docker的网络模型'
subtitle: '我的 Docker 笔记'
date: 2023-08-29 19:50:00 +0800
categories: 云原生
author: 月梦
cover: 'https://s21.ax1x.com/2024/03/11/pF6am9I.png'
cover_author: 'Srija Anaparthy'
cover_author_link: 'https://medium.com/@srijaanaparthy?source=post_page-----b157f029fb3a--------------------------------'
tags: 
- Docker 
- 云原生 
pin: true
---

`Docker` 提供了四种网络模型，其中默认的网络模型是 **Bridge** 模型，也叫网桥模型。

### 网桥
网桥(network bridge)，又称桥接器，是一种网络装置，负责网络桥接。网桥将网络的多个网段在数据链路层连接起来。与路由器不同的是，网桥将两个独立的网络连接起来，就如同单一网络。网桥可以认为是交换机。

### Bridge模型
当 `Docker` 进程启动的时候，会在宿主机上创建一个名为 `docker0` 的虚拟网桥，此主机上启动的 `docker` 容器会连接到这个虚拟网桥上。`Docker` 运行的时候会从 `docker0` 分配一个ip，并设置 `docker0` 的ip地址为容器的默认网关。在主机创建一对虚拟网卡，`veth pair` 设备，它是成对出现的，用于解决网络命名空间之间的隔离，`Docker` 将 `veth pair` 设备的一端放在新创建的容器中，并命名为 `eth0` 作为容器的网卡，另一端放在主机中，以 `vethxxx` 这样的方式命名，并将这个网络设备加入到 `docker0` 网桥中。如果容器内程序想要访问外网服务，则需要通过 `SNAT` 机制，将来自容器内部的源ip替换成宿主机ip。如果外部程序想要访问容器内服务，则需要在启动容器时做端口绑定（也就是通过 `-p` 参数）。  


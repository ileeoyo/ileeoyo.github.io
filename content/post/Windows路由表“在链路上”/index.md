---
title: "Windows路由表“在链路上”"
date: 2020-05-25T19:30:19+08:00
draft: false
tags: ["路由表","网络"]
categories: ["网络"]
typora-root-url: ./
typora-copy-images-to: ./
---

在Windows路由表中，经常会看到网关显示“在链路上”，但是有的网关显示的却是具体的ip地址，在链路上什么意思呢？

```cmd
IPv4 路由表
===========================================================================
活动路由:
           网络目标          网络掩码              网关                接口  跃点数
          0.0.0.0          0.0.0.0    25.255.255.254        10.10.10.20  10034
          0.0.0.0          0.0.0.0       192.168.1.1      192.168.1.108     35
         10.8.0.0    255.255.255.0            在链路上          10.8.0.2    281
         10.8.0.2  255.255.255.255            在链路上          10.8.0.2    281
       10.8.0.255  255.255.255.255            在链路上          10.8.0.2    281
       10.10.10.0    255.255.255.0            在链路上       10.10.10.20    291
      10.10.10.20  255.255.255.255            在链路上       10.10.10.20    291
     10.10.10.255  255.255.255.255            在链路上       10.10.10.20    291
        127.0.0.0        255.0.0.0            在链路上         127.0.0.1    331
        127.0.0.1  255.255.255.255            在链路上         127.0.0.1    331
  127.255.255.255  255.255.255.255            在链路上         127.0.0.1    331
      169.254.0.0      255.255.0.0            在链路上    169.254.34.172    281
   169.254.34.172  255.255.255.255            在链路上    169.254.34.172    281
  169.254.255.255  255.255.255.255            在链路上    169.254.34.172    281
      192.168.1.0    255.255.255.0            在链路上     192.168.1.108    291
    192.168.1.108  255.255.255.255            在链路上     192.168.1.108    291
    192.168.1.255  255.255.255.255            在链路上     192.168.1.108    291
      192.168.2.0    255.255.255.0          10.8.0.1           10.8.0.2    281
        224.0.0.0        240.0.0.0            在链路上         127.0.0.1    331
        224.0.0.0        240.0.0.0            在链路上    169.254.34.172    281
        224.0.0.0        240.0.0.0            在链路上          10.8.0.2    281
        224.0.0.0        240.0.0.0            在链路上     192.168.1.108    291
        224.0.0.0        240.0.0.0            在链路上       10.10.10.20    291
  255.255.255.255  255.255.255.255            在链路上         127.0.0.1    331
  255.255.255.255  255.255.255.255            在链路上    169.254.34.172    281
  255.255.255.255  255.255.255.255            在链路上          10.8.0.2    281
  255.255.255.255  255.255.255.255            在链路上     192.168.1.108    291
  255.255.255.255  255.255.255.255            在链路上       10.10.10.20    291
===========================================================================
永久路由:
  无
```

“在链路上”英文On-Link，字面意思就是在链路上。表示不需要通过路由器转发，可以直接与其通信的意思。显示在链路上，表示该条路由表的网关IP和接口的IP是一样的，由本机接口直接决定数据包的去向，无需其他路由中转。一般都是目标的ip与本机接口IP在同一个网段。同一个网段，自然不需要路由转发，可以直接与同网段的IP进行通信，那么网关自然是自己本身（自己接口的ip）。广播和组播`255.255.255.255`,`224.0.0.0`例外，虽然与接口ip不是同一个网段，但是也不需要其他路由器转发，同样由自己处理

1. 反例，非”在链路上“

   有路由表：` 192.168.2.0  255.255.255.0  10.8.0.1  10.8.0.2  281`

   由于目标`192.168.2.0/24`与接口ip`10.8.0.2`不在同一个网段，无法直接通信，需要路由转发，所以需要指定网关，指定发往`192.168.2.*`的数据包的下一跳，由下一跳网关决定数据包的去向。这里我的网关是`10.8.0.1`。

   同理：

   `0.0.0.0  0.0.0.0  25.255.255.254  10.10.10.20  10034`和`0.0.0.0  0.0.0.0  192.168.1.1  192.168.1.108 35`也都指定网关，由网关做路由转发，决定数据包的去向

2. 正例，在链路上

   有路由表：`10.8.0.0  255.255.255.0  在链路上  10.8.0.2  281`

   目标`10.8.0.8/24`与接口`10.8.0.2`是同一个网段，同个局域网内。假如一个发往`10.8.0.3`的数据包，由本机接口`10.8.0.2`处理，由于是同一个网段，所以本机接口`10.8.0.2`是知道包的去向的，无需路由转发，可以直接发往目标`10.8.0.3`。所以这里指定网关“在链路上”，意思就是自己这个接口本身就是网关，接口ip即是网关ip，由本机接口直接决定数据包的去向。

   
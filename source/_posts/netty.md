---
title: netty
tags:
  - nio
  - netty
  - 通信
categories:
  - netty
  - java nio
date: 2019-10-09 15:58:49
---

## java nio
1. buffer
   2. ByteBuffer position, limit, capacity, flip切换读写模式
2. channel
   1. 与buffer交互，读操作：将channel数据填充到buffer，channel.read(buffer)，写操作：将buffer数据写入channel，channel.write(buffer)
3. selector 选择器or多路复用器 非阻塞
   1. 一个线程管理多个 Channel





## tomcat nio

流程：

1. 指定 Protocol，初始化相应的 Endpoint，我们分析的是 NioEndpoint；
2. init 过程：在 NioEndpoint 中做 bind 操作；
3. start 过程：启动 worker 线程池，启动 1 个 Acceptor 和 2 个 Poller，当然它们都是默认值，可配；
4. Acceptor 获取到新的连接后，getPoller0() 获取其中一个 Poller，然后 register 到 Poller 中；
5. Poller 循环 selector.select(xxx)，如果有通道 readable，那么在 processKey 中将其放到 worker 线程池中。

![tomcat-nio](../../image/tomcat-nio.png)


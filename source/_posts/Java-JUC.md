---
title: Java-JUC
tags:
  - null
categories:
  - null
date: 2020-04-01 18:15:34
---



# Java juc源码笔记合集

[TOC]

这是一个Java juc源码笔记合集。



# 队列同步器AQS

AbstractQueuedSynchronizer抽象队列同步器。

Lock接口提供加锁、解锁的统一入口，是面向锁使用者的。

自定义锁实现时，实现Lock接口，内部类继承AQS，实现AQS的`tryAcquire`/`tryRelease`（或`tryAcquireShared`/`tryReleaseShared`），完成具体的锁获取逻辑，比如是否公平、对同步状态的变更等，是面向锁开发者的。

AQS封装了排队（什么时候阻塞、什么时候去获取锁）的逻辑。



## 公平与非公平

公平与非公平，是针对非排队的线程而言的。

对排队中的线程，都是公平的，需要按照队列顺序，当前驱节点是head时，才有资格`tryAcquire`，即尝试获取锁。

非公平是指，在队列中的线程刚被唤醒、还没来得及获取到锁时，不在排队中的、其他的线程可能先获取到了锁。以ReentrantLock的nonfaireSync为例，只要当前线程看到state为0、且cas成为1，当前线程就获取到了锁。



## `acquire`流程

`acquire`是AQS提供给Lock的加锁入口。


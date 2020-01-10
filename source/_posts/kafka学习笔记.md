---
title: kafka学习笔记
tags:
  - null
categories:
  - null
date: 2020-01-08 18:31:02
---

# 深入理解Kafka

——核心设计与实践原理 朱忠华



## 第4章 主题与分区

主题作为消息的归类，可以再细分为一个或多个分区，分区可看做对消息的二次归类。

分区为kafka提供了可伸缩性、水平扩展的功能，还通过副本机制提供数据冗余来提高数据可靠性。

主题和分区都是逻辑上的概念。分区有一个至多个副本，每个副本对应一个日志文件，每个日志文件对应一个或多个日志分段（LogSegment），每个日志分段细分为索引文件、日志存储文件和快照文件等。



### 4.1 主题的管理



```shell
bin/kafka-topics.sh --zookeeper locakhost:2181/kafka --create --topic topic-create --partitions 4 --replication-factor 2
```

（启动三个broker/节点node的集群后）创建分区数为4、副本因子为2的主题，会在3个节点上共创建8个文件夹=分区数4*副本因子2

副本与日志一一对应，每个副本对应一个命名形式如`<topic>-<partition>`的文件夹

同一个分区的多个副本必须分布在不同的broker中，才能提供有效的数据冗余

![image-20200109120448417](/github/northernw.github.io/image/image-20200109120448417.png)





### 4.2 初识KafkaAdminClient

### 4.3 分区的管理

只有leader副本对外提供读写服务，follower副本只负责在内部进行消息的同步。





## 第5章 日志存储

### 5.1 文件目录布局

物理上的形式：

1. 日志（对应于一个副本）Log是一个文件夹

2. 日志段LogSegment是一个日志文件和两个索引文件，以及可能的其他文件（比如以“.txnindex”为后缀的事务索引文件）

   ![image-20200109143542736](/github/northernw.github.io/image/image-20200109143542736.png)

   ![image-20200110115632764](/github/northernw.github.io/image/image-20200110115632764.png)

### 5.2 日志格式的演变

### 

v1比v0在RECORD中加了timestamp字段，占8B(字节)

|                                 | v0   | V1   |
| ------------------------------- | ---- | ---- |
| LOG_OVERHEAD日志头部            | 12B  | 12B  |
| RECORD_OVERHEAD_V0 消息最小长度 | 14B  |      |
| RECORD_OVERHEAD_V1              |      | 22B  |





### 5.3 日志索引

#### 5.3.1 偏移量索引

（1）relativeOffset：相对偏移量，相对于baseOffset（索引文件文件名）的偏移量，4个字节 

（2）position：物理地址，消息在日志分段文件中对应的物理地址，4个字节

![image-20200110114915258](/github/northernw.github.io/image/image-20200110114915258.png)

查找时，先通过**二分法**在偏移量索引中找到不大于给定偏移量的最大索引项，再从对应的日志分段的物理位置开始**顺序查找**给定偏移量对应的消息。

用跳跃表的结构，查找对应的偏移量索引文件。



#### 5.3.2时间戳索引

每个索引项12字节

（1）timestamp：当前日志分段最大的时间戳。-----没太理解？

（2）relativeOffset：时间戳所对应的消息的相对偏移量。

![image-20200110144142268](/github/northernw.github.io/image/image-20200110144142268.png)

时间戳索引文件中包含若干时间戳索引项，每个追加的时间戳索引项中的timestamp 必须大于之前追加的索引项的 timestamp，否则不予追加。



这部分没太看懂 = =

![image-20200110144417507](/github/northernw.github.io/image/image-20200110144417507.png)





### 5.4 日志清理

#### 5.4.1 日志删除

有专门的日志删除任务，定期（默认为5分钟，300000ms）检测和删除不符合保留条件的日志分段文件

3种保留策略：

1. 基于时间：默认情况下日志分段文件的保留时间为7天
2. 基于日志大小
3. 基于日志起始偏移量

#### 5.4.2 日志压缩

LogCompaction对于有相同key的不同value值，只保留最后一个版本。

如果应用只关心key对应的最新value值，则可以开启Kafka的日志清理功能。





### 5.5 磁盘存储







# Apache Kafka

官网：https://kafka.apache.org/intro

中文官网：http://kafka.apachecn.org/intro.html

## 介绍

Apache kafka是一个分布式流处理平台。

特性：

1. 发布订阅流式记录，类似于消息队列或企业级消息系统。
2. 存储流式的记录，有较好容错性。
3. 在流式记录产生时就可以进行处理。



应用：

1. 构造实时流数据管道（消息队列）
2. 构建实时流式应用程序，转换或响应流数据（流处理，kafka stream topic和topic---流处理不太了解）



概念：

1. kafka运行在集群（一个或多个服务器）上，可跨越多个数据中心
2. kafka集群通过topic对消息分类
3. 每条消息包含一个key，一个value，一个timestamp



四个核心api

1. producer api 允许发布一串流式的数据到一个或多个主题

2. Consumer api 允许应用程序订阅一个或多个主题，并对发布给他们的流式数据进行处理

3. streams api 允许一个应用程序作为流处理器，消费一个或多个主题产生的输入流，然后生产一个输出流到一个或多个主题中去。在输入输出流中进行有效的转换。

4. connector api--不太了解
   1. 原文：允许构建并运行可重用的生产者或者消费者，将Kafka topics连接到已存在的应用程序或者数据系统。比如，连接到一个关系型数据库，捕捉表（table）的所有变更内容。



## Topics和日志

kafka中的topics总是多订阅者模式，一个topic可以有一个或多个消费者来订阅数据。

对每个主题，kafka都会维护一个分区日志：

![image-20200108194432350](/github/northernw.github.io/image/image-20200108194432350.png)

每一个topic有多个分区，**每个分区都是有序且顺序不可变的记录集**。

分区表示记录顺序的标号叫offset（偏移量），唯一标识分区中的每一条记录。

记录不会因为被消费过就被集群删掉，而是通过保留期限这个参数控制的。保留期限内可随时被消费，过期后会被抛弃并释放磁盘空间。kafka的性能在数据大小方面是稳定的，所以长时间存储数据不是问题。

另一反面，消费者唯一保持的元数据是offset，即消费在log中的位置，由消费者自己控制。（可以任意修改offset的值，消费已消费的、或者跳过某些消息）

日志分区的作用：

1. 当日志大小超过单台服务器的限制，允许日志进行扩展。（可以处理无限量的数据）
2. 可以作为并行的单元集



# Apache Kafka工作原理介绍--博客

原文地址：https://www.ibm.com/developerworks/cn/opensource/os-cn-kafka/index.htmlhttps://www.ibm.com/developerworks/cn/opensource/os-cn-kafka/index.html

## 术语

Broker: kafka集群包含一个或多个服务器，服务器称为broker

Topic: 每条发布到kafka集群的消息都有一个类别，这个类别称为topic（物理上可能分开存储，用户不必关心）

Partition：物理上的概念，每个topic包含一个或多个Partition

Producer：生产者，发布消息到broker

Consumer：消费者，从broker读取消息

Consumer Group：每个Consumer属于一个特定的组（可为每个Consumer指定group name，不指定group name则属于默认的group）



kafka交互流程

每个消息只会发送给群组中的一个消费者，有相同键值的消息都会被确保发给这一个消费者。





## 问题

1. Consumer group做什么用？为什么要设计组这个概念？
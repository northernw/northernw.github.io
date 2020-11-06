---
title: 技术栈Notes
tags:
  - null
categories:
  - null
date: 2020-10-26 14:36:11
---



# 第一阶段

## 一、MyBatis

### 1. Mybatis框架分析

（1）mybatis框架架构图

（2）整体执行流程图

（3）sqlSession执行流程图

### 2. MyBatis源码分析

（1）Config文件加载流程

（2）mapper文件加载流程

（3）SQLSource创建流程

（4）获取BoundSql流程

（5）参数映射流程

（6）结果集映射流程

### 3. 设计模式

构造者模式

简单工厂模式

工厂方法模式

抽象工厂模式

单例模式

### 4. 手写Mybatis

实现配置文件加载流程

实现封装jdbc的执行流程





## 二、Spring

### 1. 核心模块介绍

（1）核心容器模块

core模块

beans模块

context模块

expression模块

（2）AOP和设备模块

（3）数据访问及集成模块

（4）web模块

（5）报文消息模块

（6）test测试模块

### 2. 核心接口讲解

（1）BeanFactory接口体系

（2）BeanDefinition接口体系

（3）ApplicationContext接口体系

### 3. 源码分析

（1）IOC源码解析

1. IOC初始化流程
2. BeanDefinition加载注册流程
3. DI依赖注入流程

（2）aop流程

1. aop标签解析流程
2. AspectJAwareAdvisorAutoProxyCreator类<u>的作用</u>
3. AspectJExpressionPointcut类<u>的作用</u>
4. AspectJPointcutAdvisor类的作用
5. 动态代理对象创建流程
6. AspectJAwareAdvisorAutoProxyCreator类<u>的实现</u>
7. AspectJExpressionPointcut类<u>的实现</u>

（3）tx流程

1. spring事务的实现原理
2. PlatformTransactionManager的接口体系
3. TransactionStatus接口



### 4. 设计模式

（1）创建型

1. 简单工厂模式
2. 工厂方法模式
3. 抽象工厂模式
4. 单例模式
5. 原型模式

（2）结构型

1. 代理模式
   1. jdk动态代理模式
   2. cglib动态代理模式
2. 装饰模式
3. 组合模式

（3）行为型

### 5. 手写框架

（1）IOC模块

1. 实现IOC初始化流程
2. 实现BeanDefinition的加载注册
3. 实现DI依赖注入

（2）AOP模块

### 6. 常见分析

1. BeanFactory和FactoryBean的区别
2. BeanFactoryPostProcessor和BeanPostProcessor的区别
3. 循环依赖和循环构造问题如何解决



## 三、SpringMVC

### 1. 架构分析

（1）11步执行流程图

（2）六大组件介绍

1. DispatcherServlet
2. HandlerMapping
3. HandlerAdapter
4. Handler
5. ViewResolver
6. View

### 2. 源码分析

（1）DispatcherServlet初始化流程

1. HandlerMapping初始化流程
2. HandlerAdapter初始化流程

（2）DispatcherServlet执行流程

（3）HandlerMapping执行流程

1. 解析@Controller和@RequestMapping流程

（4）HandlerAdapter执行流程

1. 参数设置流程
2. 结果映射流程

### 3. 手写springmvc

源码分析里的几个组件和流程的实现





## 四、MySQL

### 1. SQL语法顺序和解析顺序的理解

### 2. MySQL架构分析和执行流程分析

#### （1）逻辑架构图

连接器

服务管理

连接池

SQL接口

解析器

查询优化器

查询缓存

可插拔存储引擎：MyISAM、InnoDB、Memory

#### （2）执行流程图

1. 简单执行流程
2. 详细执行流程

#### （3）物理存储结构

1. 日志文件
   1. 错误日志
   2. 二进制日志
   3. 通用查询日志
   4. 重做redo日志
   5. 回滚undo日志
   6. 中继日志
2. 数据文件
   1. InnoDB数据文件
   2. MyISAM数据文件

### 3. 索引篇

#### （1）索引基础

1. 索引介绍
   1. 什么是索引
   2. 索引优势
   3. 索引劣势

2. 索引分类
   1. 单列索引
   2. 组合索引
   3. 全文索引
   4. 空间索引

3. 索引使用

   1. 索引创建
   2. 索引删除
   3. 索引查看

   

#### （2）索引原理

1. 索引存储结构
   1. B树与B+树
   2. 非聚集索引（MyISAMEMStore索引）
   3. 聚集索引（InnoDB）
2. 多用组合索引
   1. 组合索引的优势
   2. 最左前缀原则
3. 索引使用场景
   1. 使用索引情况
   2. 不使用索引情况
4. 索引失效分析
   1. 查看执行计划
      1. explain查看执行计划
      2. 参数说明
         1. select_type
            1. simple
            2. primary
            3. subquery
            4. union
         2. type
            1. const
            2. eq_ref
            3. ref
            4. range
            5. index
            6. all
         3. extra
            1. using index
            2. using where
            3. using index condition
               1. ICP的理解
               2. index filter的理解
            4. using filesort
   2. 多个索引使用案例分析



### 4. 锁和事务篇

#### （1）锁的介绍

1. 表级锁
2. 行级锁
   1. 行锁
   2. MDL元数据锁

#### （2）行锁原理分析

1. 简单SQL的加锁分析
   1. RC级别下的主键索引、唯一索引、非唯一索引、无索引分析
   2. RR级别下的主键索引、唯一索引、无唯一索引、无索引分析
2. 复杂SQL的加锁分析
   1. where条件如何拆分
      1. index key
      2. index filter
      3. table filter
3. 死锁原理分析
   1. 两个session的两条SQL产生死锁分析
   2. 两个session的一条SQL产生死锁分析

#### （3）事务流程分析

1. 回滚流程undo
2. 重做流程redo

#### （4）InnoDB架构分析

1. 架构图分析
2. 内存结构分析
   1. Buffer Pool
      1. data page和index page
      2. insert buffer
      3. adaptive hash index
      4. lock info
      5. data dictionary
   2. Redo log buffer
   3. double write
3. 磁盘文件分析
   1. 系统表空间和用户表空间文件
   2. 重做日志文件和归纳文件（？）
   3. 重做日志的落盘机制

#### （5）InnoDB一致性非锁定读

1. 一致性非锁定读的机制
2. MVCC（多版本并发控制）原理
3. InnoDB的MVCC实现

#### （6）InnoDB事务分析

1. 原子性、一致性、持久性原理分析
2. 隔离性原理分析
   1. 事务并发问题理解
   2. 当前读和快照读
   3. 一致性非锁定读理解
   4. InnoDB的MVCC实现



### 5. 性能分析篇

（1）性能分析思路

（2）慢查询日志分析

1. 何时开启慢查询日志
2. 设置慢查询日志超时时间
3. 分析慢查询日志的工具

（3）查询计划分析

（4）profile性能分析

### 6. 性能优化篇

（1）服务器层面优化

1. innodb_buffer_pool_size设置
2. 内存预热
3. innodb_log_file_size设置
4. 选择SSD磁盘提高读写能力

（2）SQL设计层面优化

1. 中间表的设计
2. 冗余字段的设计
3. 拆表字段
4. 拆表数据（分库分表）

（3）SQL语句优化

1. limit优化
2. 索引优化
   1. 如何创建索引并正确使用组合索引
   2. order by group by与索引设计的关联
3. 其他优化项

### 7. 主从复制和读写分离集群

（1）主从复制集群

1. 主从复制原理
2. binlog和relay日志
3. 主从复制实践

（2）读写分离集群

1. 原理分析
2. 读写分离实践

### 8. 分库分表篇

（1）分库分表策略

1. 数据切分方案
2. 数据切分规则
3. 收切分原则
4. 分库分表要解决的问题
   1. 分布式事务问题
   2. 分布式主键ID问题
   3. 跨库join问题
   4. 跨库count、order by、group by问题

（2）MyCat集群

1. 架构介绍
2. 核心概念介绍
3. 十种常见分片规则
4. MyCat集群搭建与分库分表应用
5. MyCat读写分离方式设置
6. 用户购物下单实践





## 五、Redis

### 1. Redis五种数据类型及使用场景分析

### 2. Redis事务

（1）Redis事务分析

（2）事务失败的处理

### 3. 持久化原理及性能分析

（1）rdb方式

1. 快照触发时机
2. 设置快照规则
3. 快照实现原理
4. 优缺点分析

（2）aof方式

1. 同步磁盘数据分析
2. aof重写原理分析
3. 文件损坏如何恢复

（3）如何选择rdb和aof

### 4. 主从复制

原理分析

主从配置的实现原理

1. 全量同步
2. 增量同步

### 5. 哨兵机制

哨兵进程的作用分析

故障判断原理分析

1. ODOWN
2. SDOWM
3. 法定人数理解

### 6. cluster集群

Redis集群策略

架构分析

容错机制

redis cluster集群搭建

数据迁移

### 7. 与lua整合

如何编写包含redis api的lua脚本

Redis整合lua脚本

### 8. Redis消息模式

队列模式

发布订阅者模式

### 9. 分布式锁

分布式锁的实现方式

注意事项

分布式锁实战与高并发测试

### 10. 常见缓存问题与解决方案

带来的问题都是：加大数据库压力

#### 缓存穿透

现象：数据库和缓存中都不存在的数据，查询这种不存在的数据的现象

解决方案：

1. 布隆过滤器 BloomFilter（有比较小的误判的概率，会认为一个不存在的数据存在，即允许这样的请求执行）
2. 缓存数据的ID

#### 缓存击穿

现象：大量请求查询同一个key，而这个key正好失效了，就会导致大量请求打到数据库上

解决方案：在查询数据库的逻辑前加排他锁，获取锁的线程可以查询，并将结果放入缓存（释放锁后，其他线程在获取锁前再读一次缓存，double check）

#### 缓存雪崩

现象：大量的key在同一时间失效，或者缓存宕机了

解决方案：

1. 使用缓存集群，降低服务宕机的概率
2. 本地缓存+限流&降级

#### 热点数据集中失效

现象：一般会给key设置失效时间，热点数据请求比较多，会都打到数据库上

解决方案：

1. 热点数据和非热点数据设置不同的失效时间
2. 采用缓存击穿的方案，查询数据库加排他锁
3. 设置为永不失效，采用定时任务对快失效的热点数据进行更新失效时间，进行续租

#### 缓存双写一致性





## ~~六、MongoDB~~





# 第二阶段

## 一、Zookeeper

### 1. 简介

重要概念讲解

Paxos算法

ZAB协议

CAP原则

### 2. 源码解析

Watcher核心机制

Leader选举

### 3. 应用场景

注册中心

分布式锁

分布式队列

负载均衡

配置维护

命名服务

DNS服务

分布式同步

集群管理





## 二、网络通信

### RPC

原理与本质

1. RPC理论
2. RPC基于序列化信息通信

RPC解决什么问题

1. 解决SOA编程模式问题……？

RPC实例实战

1. RPC基于socket相连
2. RPC基于动态代理调用透明化

RPC模块化演进

1. 通过设计模式实现RPC模块单一职责

### Netty

IO模型原理

1. NIO事件驱动流性能优化
2. BIO和OIO堵塞流问题
3. AIO异步流使用场景

线程模型

1. NioEventLoop线程模型



驱动模式

1. ServerBootstrap启动原理
2. Bootstrap启动原理



Codec框架

1. tcp黏包、拆包
2. encode编码器
3. decode解码器



通道

1. channelHandler
2. channelInboundHandler
   1. ChannelInitializer
   2. ByteToMessageDecoder
3. channelOutboundHandler
   1. MessageToMessageEncoder
   2. LineEncoder



责任链模式 channelPipeline



上下文 channelHandlerContext



配置构建模式 channelConfig



内部类 unsafe



主从模型 BossGroup&WorkerGroup



零拷贝 zero-copy



字节容器 ByteBuf原理



Netty实战 websocket聊天



## 三、Dubbo

### 入门

Dubbo分布式服务模块划分



### 高可用

容错机制

服务降级

服务限流

服务暴露延迟

结果缓存应用

多版本控制

多注册中心



### 高级应用

Dubbo负载均衡策略与自定义实现

仅订阅与仅注册

提供者的异步调用、异步执行

Dubbo源码解析

1. Dubbo的SPI
2. Dubbo对spring配置文件的加载与解析
3. provider的服务暴露
4. Consumer的服务消费



## ~~四、Nginx~~

## 五、消息中间件

消息中间件在分布式架构中的应用场景：业务解耦/最终一致性/广播/错峰流控



JMS：Java Message Service，Java消息服务。试图通过提供公共Java API，隐藏单独MQ产品供应商提供的实际接口，跨越壁垒，解决互通问题。

AMQP：Advanced Message Queue Protocol，高级消息队列，2006年提出。是应用层协议的一个开放标准，基于此协议的客户端与消息中间件可传递消息，不受产品、开发语言等条件的限制。



四个项目都是开源的

| 产品     | 启动时间&开发者 ，开源时间    | 语言       | 吞吐量 | 时效性 | 可用性             | 消息可靠性           | 说明                                                         |
| -------- | ----------------------------- | ---------- | ------ | ------ | ------------------ | -------------------- | ------------------------------------------------------------ |
| ActiveMQ | 2004LogicBlaze，2007Apache    | Java       | 万级   | 毫秒级 | 高，基于主从架构   | 有较低的概率丢失数据 | 早期活跃、成熟产品，现在用的少。主要用于解耦和异步，较少在大规模吞吐的场景中使用。现在社区不活跃 |
| RabbitMQ | 2007RabbitMQ Technologies Ltd | erlang     | 万级   | 微秒级 | 高，基于主从架构   |                      | erlang开发，并发能力强，性能极好，延时低。社区活跃           |
| RocketMQ | 2012阿里开源，2016Apache      | Java       | 十万级 | 毫秒级 | 非常高，分布式架构 | 可以做到0丢失        | 大规模吞入，性能好，分布式扩展方便。社区活跃一般             |
| Kafka    | 2011LinkedIn，2012Apache      | Scala+Java | 十万级 | 毫秒级 | 非常高             | 可以做到0丢失        | 超高吞吐量，极高的可用性和可靠性，分布式任意扩展。大数据领域实时计算、日志采集的事实标准 |



RabbitMQ概念

1. 生产者和消费者
2. 队列
3. 交换器、路由键、绑定



Kafka

1. 基于zookeeper实现高可用
2. 消息处理过程剖析
3. 副本机制与选举原理
4. 





## 六、SpringBoot

## 七、SpringCloud

## ~~反应式Web开发框架Webflux~~

# 第三阶段

## 一、FastDFS

## 二、ElasticSearch

## ~~互联网电商项目~~
---
title: Redis学习笔记
tags:
  - null
categories:
  - null
date: 2020-05-29 16:54:04
---



# redis环境

[docker安装redis](https://www.runoob.com/docker/docker-install-redis.html)

```
docker pull redis:latest
docker run -itd --name redis-test -p 6379:6379 redis
docker exec -it redis-test /bin/bash
redis-cli
```





# 数据结构

[the little redis book](https://github.com/JasonLai256/the-little-redis-book/blob/master/cn/redis.md)

## 字符串（Strings）

`set``

`strlen <key>`

`getrange <key> <start> <end>`

`append <key> <value>`

计数`incr`  `incrby` `decr` `decrby`

存储对象（简单或复杂）和计数

位图`setbit` `getbit`

## 散列（Hashes）

`hset` `hget`

相关的操作还包括在同一时间设置多个域、同一时间获取多个域、获取所有的域和值、列出所有的域或者删除指定的一个域：

```shell
hmset users:goku race saiyan age 737
hmget users:goku race powerlevel
hgetall users:goku
hkeys users:goku
hdel users:goku age
```

## 列表（Lists）

对于一个给定的关键字，列表数据结构让你可以存储和处理一组值。你可以添加一个值到列表里、获取列表的第一个值或最后一个值以及用给定的索引来处理值。列表数据结构维护了值的顺序，提供了基于索引的高效操作。

```
lpush newusers goku
ltrim newusers 0 50
```

```
keys = redis.lrange('newusers', 0, 10)
redis.mget(*keys.map {|u| "users:#{u}"})
```



## 集合（Sets）

集合数据结构常常被用来存储只能唯一存在的值，并提供了许多的基于集合的操作，例如并集。集合数据结构没有对值进行排序，但是其提供了高效的基于值的操作。

使用集合数据结构的典型用例是朋友名单的实现：

```
sadd friends:leto ghanima paul chani jessica
sadd friends:duncan paul jessica alia
```

而且，我们可以查看两个或更多的人是不是有共同的朋友：

```
sinter friends:leto friends:duncan
```

甚至可以在一个新的关键字里存储结果：

```
sinterstore friends:leto_duncan friends:leto friends:duncan
```

## 有序集合（Sorted Sets）

最后也是最强大的数据结构是分类集合数据结构。

提供了排序功能 sorting and ranking

```
zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir
```


对于`duncan`的朋友，要怎样计算出标记（score）为90或更高的人数？

```
zcount friends:duncan 90 100
```

如何获取`chani`在名单里的倒排索引值（rank）？

```
zrevrank friends:duncan chani
```



# 时间复杂度

常数时间复杂度O(1)

对数时间复杂度O(log(n))

线性时间复杂度O(n)

二次时间O(n^2)

指数时间O(c^n)



# Round Trips and Pipelining

数据往返？批量数据？

mget命令，接受多个关键字，然后返回值：

```
keys = redis.lrange('newusers', 0, 10)
redis.mget(*keys.map {|u| "users:#{u}"})
```

或者是sadd命令，能添加一个或多个成员到集合里：

```
sadd friends:vladimir piter
sadd friends:paul jessica leto "leto II" chani
```

支持流式请求

```
redis.pipelined do
  9001.times do
	redis.incr('powerlevel')
  end
end
```





# 事务

Redis是单线程运行，每个命令都具有原子性

[为什么选择单线程模型](https://zhuanlan.zhihu.com/p/52600663)

首先要执行`multi`命令，紧随其后的是所有你想要执行的命令（作为事务的一部分），最后执行`exec`命令去实际执行命令，或者使用`discard`命令放弃执行命令。Redis的事务功能保证了什么？

- 事务中的命令将会按顺序地被执行
- 事务中的命令将会如单个原子操作般被执行（没有其它的客户端命令会在中途被执行）
- 事务中的命令要么全部被执行，要么不会执行

```
multi
hincrby groups:1percent balance -9000000000
hincrby groups:99percent balance 9000000000
exec
```

但这样并不能避免并发问题，因为可能有多个Redis客户端在
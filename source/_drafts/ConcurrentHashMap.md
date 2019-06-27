---
title: ConcurrentHashMap
date: 2019-06-23 17:18:42
tags:
---
## 问题
1. helpTransfer 帮助扩容，how？
2. 扩容的实现？
3. 有哪些主要filed和method？
4. 和1.7的区别？
5. 如何实现并发？加锁解锁？
6. key.hashCode()只提供若干低位信息
7. 节点hash 为0或者小于0的情况
8. ((long)i << ASHIFT) + ABASE 作用？



## 重要的属性
### sizeCtl
```java
    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     * 负数：表示在初始化表或扩容
     * -1: initTable
     * -(1+n): n表示在处理扩容的线程数
     * table为null时，值为将要创建的表的大小
     * 0为初始值
     */
    private transient volatile int sizeCtl;
```


## 散列

key.hashCode()只提供若干低位信息

```java
    static final int HASH_BITS = 0x7fffffff;
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```


## 扩容

原表中不同桶上的结点，在新表上一定不会分配到相同位置的槽上。
helpTransfer
transfer


## put方法

put方法做了以下几点事情：
1）如果没有初始化就先调用initTable()方法来进行初始化过程
2）如果没有hash冲突就尝试CAS方式插入
3）如果还在进行扩容操作就先帮助其它线程进一起行扩容
4）如果存在hash冲突，就加锁来保证put操作的线程安全



## remove方法

cv是什么？

setTabAt(tab, i, e.next); 作用？

validated 作用？

```java
final V replaceNode(Object key, V value, Object cv)
```





## 重要属性

负数代表正在进行初始化或扩容操作

-1代表正在初始化

-N 表示有N-1个线程正在进行扩容操作

正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，这一点类似于扩容阈值的概念。还后面可以看到，它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。

## 位运算备忘
摘自 https://www.jianshu.com/p/eb9ab4211163
```java
<< : 左移运算符，num << 1,相当于num乘以2  保留符号位，低位补0
>> : 右移运算符，num >> 1,相当于num除以2  保留符号位，高位补0
>>> : 无符号右移，忽略符号位，空位都以0补齐
 % : 模运算 取余
^ :   位异或 第一个操作数的的第n位与第二个操作数的第n位不同，那么结果的第n为也为1，否则为0
 & : 与运算 第一个操作数的的第n位与第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0
 | :  或运算 第一个操作数的的第n位与第二个操作数的第n位 只要有一个是1，那么结果的第n为也为1，否则为0
 ~ : 非运算 操作数的第n位为1，那么结果的第n位为0，反之，也就是取反运算（一元操作符：只操作一个数）
```




## 参考

- [Java 8 中 ConcurrentHashMap工作原理的要点分析](https://www.bbsmax.com/A/ZOJPOX7xzv/)
- [java-并发-ConcurrentHashMap高并发机制-jdk1.8](https://blog.csdn.net/jianghuxiaojin/article/details/52006118)
- [探索jdk8之ConcurrentHashMap 的实现机制](https://www.cnblogs.com/huaizuo/p/5413069.html)
- [Java中Unsafe类详解](https://www.cnblogs.com/mickole/articles/3757278.html)
- [为并发而生的 ConcurrentHashMap（Java 8）](https://www.cnblogs.com/yangming1996/p/8031199.html)
- [java8集合框架(三)－Map的实现类（ConcurrentHashMap）](http://wuzhaoyang.me/2016/09/05/java-collection-map-2.html)
- [并发编程——ConcurrentHashMap#transfer() 扩容逐行分析](https://www.jianshu.com/p/2829fe36a8dd)
- [ConcurrentHashMap 源码阅读小结](https://www.jianshu.com/p/29d8e66bc3bf)

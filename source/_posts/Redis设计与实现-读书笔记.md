---
title: Redis设计与实现-读书笔记
tags:
  - null
categories:
  - null
date: 2020-06-11 17:28:50
---



# 数据结构与对象

1. 简单动态字符串

   SDS simple dynamic string

   ads.h/adshdr

   <img src="/github/northernw.github.io/image/image-20200611191443812.png" alt="image-20200611191443812" style="zoom:25%;" />



2. 链表（双端链表）

   列表键的实现之一：列表健包含了数量较多的元素，或者列表中的元素是比较长的字符串

   adlist.h/listNode

   adlist.h/list

   ![image-20200611191731604](/github/northernw.github.io/image/image-20200611191731604.png)

   ![image-20200611191938934](/github/northernw.github.io/image/image-20200611191938934.png)

3. 字典

   哈希键的实现之一：哈希键的键值对比较多，或者键值对中元素是比较长的字符串

   dict.h/dict

   dict.h/dictht 哈希表

   ![image-20200611192134108](/github/northernw.github.io/image/image-20200611192134108.png)

   ![image-20200611192151751](/github/northernw.github.io/image/image-20200611192151751.png)

   ![image-20200611192316123](/github/northernw.github.io/image/image-20200611192316123.png)

4. 跳跃表

   有序集合键的实现之一：包含元素数量多，或者有序集合中元素的成员（member）是比较长的字符串

   redis.h/zskiplistNode

   redis.h/zskiplist

   ![image-20200611192508163](/github/northernw.github.io/image/image-20200611192508163.png)

5. 整数集合

   集合键的实现之一：集合只包含整数值元素，并且数量不多

   intset.h/intset

   ![image-20200611192614681](/github/northernw.github.io/image/image-20200611192614681.png)

6. 压缩列表

   列表键和哈希键的底层实现之一：当列表键只包含少量列表项，并且每个列表项要么是小整数，要么是长度比较短的字符串

   ziplist

   ![image-20200611193109349](/github/northernw.github.io/image/image-20200611193109349.png)

   ![image-20200611193129132](/github/northernw.github.io/image/image-20200611193129132.png)

7. 对象

   以上的数据结构都封装在对象中再使用

   包含**字符串对象、列表对象、哈希对象、集合对象和有序集合对象**这五种类型的对象

   Redis根据对象的类型来判断一个对象是否可以执行给定的命令

   还实现了基于引用计数计数的内存回收机制，对象可共享

   ![image-20200611193700470](/github/northernw.github.io/image/image-20200611193700470.png)

   type的枚举值

   ![image-20200611193737253](/github/northernw.github.io/image/image-20200611193737253.png)

   encoding属性记录了对象所使用的编码，也即是说这个对象使用了什么数据结构作为对象的底层实现

   ![image-20200611193923869](/github/northernw.github.io/image/image-20200611193923869.png)

   每种类型的对象都至少使用了两种不同的编码

   ![image-20200611194007998](/github/northernw.github.io/image/image-20200611194007998.png)

   **string: int / embstr / raw**

   **list: ziplist/ linkedlist 压缩列表或者链表**

   （ziplist字符串元素长度小于64字节，元素个数小于512个） 

   ![image-20200611194332923](/github/northernw.github.io/image/image-20200611194332923.png)

   ![image-20200611194347437](/github/northernw.github.io/image/image-20200611194347437.png)

   **hash: ziplist / ht 压缩列表或者字典**

   （ziplist键值对键和值的字符串长度都小于64字节&&键值对数量个数小于512个）

   ![image-20200611194553335](/github/northernw.github.io/image/image-20200611194553335.png)

   ![image-20200611194609549](/github/northernw.github.io/image/image-20200611194609549.png)

   **set: intset / ht 整数集合或者字典**

   （intset所有元素都是整数值&&个数小于512个）

   （使用字典时，值设置为NULL）

   ![image-20200611194904298](/github/northernw.github.io/image/image-20200611194904298.png)

   **zset: ziplist / skiplist 压缩列表或者跳跃表**

   （ziplist所有元素成员的长度小于64字节&&个数小于128个）

   ![image-20200611195011400](/github/northernw.github.io/image/image-20200611195011400.png)

   ![image-20200611195025054](/github/northernw.github.io/image/image-20200611195025054.png)

   同时使用跳跃表和字典：跳跃表实现有序集合，方便范围操作；字典方便查找成员的分值

   ![image-20200611195122748](/github/northernw.github.io/image/image-20200611195122748.png)

8. 多态

   DEL、EXPIRE等命令和LLEN等命令的区别在于，前者是基于类型的多态——一个命令可以同时用于处理多种不同类型的键，而后者是基于编码的多态——一个命令可以同时用于处理多种不同编码（比如说list的ziplist和linkedlist都实现了LLEN）。
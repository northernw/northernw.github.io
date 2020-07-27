---
title: 数据库
tags:
  - null
categories:
  - null
date: 2020-07-27 16:29:48
---

# Java

## Java基础

###### Java反射原理， 注解原理？

反射原理：在运行状态下，对于任何一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能调用它的任意方法和属性，并能改变它的属性。总结来说，反射把Java类中的各个成分映射成为一个个Java对象，并且可以进行操作。



注解原理：注解的本质是一个继承了Annotation接口的接口。

解析一个类或者方法的注解有两种形式，一是编译期扫描，如@Override，编译器会检查方法是否真的重写了父类的某个方法；二是运行期发射，虚拟机规范定义了一系列和注解相关的属性表，字段、属性或类上有注解时（被注解修饰了），会写相应信息进字节码文件，Class类中提供了一些接口用于获取注解或判断是否被某个注解修饰。

延伸阅读：[JAVA 注解的基本原理](https://www.cnblogs.com/yangming1996/p/9295168.html)



ps: Java类执行的过程/类加载过程（2-6）/类的生命周期（2-8） -- **tbc 更准确的说法**

1. 编译：Java文件编译成.class字节码文件
2. 加载：类加载器通过全限定名，将字节码加载进JVM，存储在方法区，将其转换为一个与目标类型对应的Class对象实例
3. 验证：格式（.class文件规范）验证和语义（final不能继承等）验证？
4. 准备：静态变量赋初值与内存空间，final修饰的内存空间直接赋原值（？），不是开发人员赋的初值
5. 解析：符号引用转换为直接引用，分配地址（?）
6. 初始化：先初始化父类，再初始化自身；静态变量赋值，静态代码块执行。
7. 使用
8. 卸载



###### Java中==、equals与hashCode的区别和联系

https://juejin.im/entry/59b3897b5188257e733c24eb -- 后面写的比较乱

https://juejin.im/post/5a4379d4f265da432003874c -- equals与hashCode

Java数据类型

1. 8种基本数据类型
   1. （整型）数值类型 byte short int long 1.2.4.8
   2. （浮点）数值类型 float double 4.8
   3. 字符型 char **2  **存储 Unicode 码，用单引号赋值。
   4. 布尔类型  boolean 1
2. 3种引用类型：类、接口、数组



==

比较两个数据是否相等，基本类型比较数值是否相等，引用类型比较地址是否相等。



equals()方法

Object类型定义的，比较二者==

```java
//object的equals方法
public boolean equals(Object obj) {
    return (this == obj);
}
```

想自定义对象逻辑“相等”（值相等、或内容相等）的含义时，重写equals方法。

重写equals准则：

> **自反性**：对于任何非空引用值 x，x.equals(x) 都应返回 true。

> **对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。

> **传递性**：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true， 并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。

> **一致性**：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false， 前提是对象上 equals 比较中所用的信息没有被修改。

> **非空性**：对于任何非空引用值 x，x.equals(null) 都应返回 false。



一般只判断同类型的对象

主要不要违反了对称性、传递性



hashCode()方法

```java
public native int hashCode();
```

equals的对象hashcode一定相等，hashcode相同的对象不一定equals

为什么对象的hashcode会相同？

hashcode的实现取决于jvm，比较典型的一种是基于内存地址进行哈希计算，也有基于伪随机的实现。

哈希计算会存在哈希碰撞。

https://juejin.im/entry/597937cdf265da3e114cd300



###### 谈谈final、finally、finalize的区别 -- 放一起有点奇怪

final

1. 修饰类、方法或变量
2. 修饰类：表明类不能被继承
3. 方法：禁止在子类中被覆盖（private方法会隐式被指定为final）
4. 变量：
   1. 基本数据类型的变量：数值在初始化后不能更改
   2. 引用类型的变量：初始化后不能再指向另一个对象（指向的地址不可变）



finally：

一般与try catch一起使用，无论程序抛出异常或正常执行，finally块的内容一定会被执行。

最常用的地方：通过try-catch-finally来进行类似资源释放、保证解锁等动作。



finalize

Object的protected方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。

日常开发中基本不用，也不推荐使用。Java9中被标记为deprecated! -- 不想多说

https://juejin.im/post/5b9bb81ef265da0ac2565a0f



###### java如何实现序列化的，Serialization底层如何实现的

简单说来，是将类信息和数据信息递归写成字节信息



###### java中的反射

field的赋值底层实现

以UnsafeBooleanFieldAccessorImpl为例，也是利用unsafe 偏移

ps: Unsafe工具类 static final Unsafe unsafe = Unsafe.getUnsafe();

```java
// set    
    public void set(Object obj, Object value)
        throws IllegalArgumentException, IllegalAccessException
    {
        ensureObj(obj);
        if (isFinal) {
            throwFinalFieldIllegalAccessException(value);
        }
        if (value == null) {
            throwSetIllegalArgumentException(value);
        }
        if (value instanceof Boolean) {
          // 这里
            unsafe.putBoolean(obj, fieldOffset, ((Boolean) value).booleanValue());
            return;
        }
        throwSetIllegalArgumentException(value);
    }

// get
    public Object get(Object obj) throws IllegalArgumentException {
        return Boolean.valueOf(getBoolean(obj));
    }

    public boolean getBoolean(Object obj) throws IllegalArgumentException {
        ensureObj(obj);
      // 这里
        return unsafe.getBoolean(obj, fieldOffset);
    }
```







## Java容器

###### 1. Java容器有哪些？哪些是同步容器,哪些是并发容器？

容器分两个大类，Collection和Map。Collection又分List、Set、Queue、Vector几个大类，Map有HashMap、TreeMap、LinkedHashMap、HashTable，其中，Vector、HashTable是同步容器。

并发容器一般在juc包下，有ConcurrentHashMap、CopyOnWriteArrayList等。

ps:

List: ArrayList、LinkedList

Set: HashSet、LinkedHashSet、TreeSet

Queue: LinkedList、PriorityQueue



引申：几个容器的主要方法的操作流程，容器体系结构



###### 2. ArrayList和LinkedList的插入和访问的时间复杂度？

ArrayList：插入O(n) 访问O(1)

LinkedList：插入O(1) 访问O(n)



###### HashMap在什么情况下会扩容，或者有哪些操作会导致扩容？

java8中

1. 放入新值（putValue--put/putMapEntries）后，元素个数size大于阈值threshold，会触发扩容。
2. 链表树化时，如果表长table.length小于64，会用扩容代替树化。
3. put值前，如果表长为0，会触发扩容



###### HashMap put方法的执行过程？

1. 如果table为空，或长度为0，初始化。默认loadFactor为0.75，默认capacity为16（capacity是table的长度），threshold一般为capacity*loadFactor。
2. 通过hash定位槽，如果槽为空，构造新节点赋值给槽
3. 若槽不为空，则在槽的链表或树中找到key相同的节点，替换节点值为新值；或是没有key相同的节点，就在树中或链表尾部加入新节点；若链表加入新节点后长度达到8（槽不算，aka槽下原有7个节点），则进行红黑树转化
4. 如果是新加入节点，modCount、元素个数size自增1，如果元素个数超过阈值，则进行扩容



###### Java8扩容的执行过程？

1. 计算新容量newCap和新阈值newThr（ps: 当容量已到最大值时，不再扩容；2倍扩容；）

2. 创建新的数组，赋值给table
   3. 将键值对重新映射到新数组上
      1. 如果无链表，直接根据`hash&(newCap-1)`定位
   4. 如果是树节点，委托红黑树来拆分和重新映射
   5. 为链表，根据`hash&oldCap`的值分成0、1两组，映射到j和j+oldCap（0低位，1高位）（**链表顺序不变**）



###### HashMap概述

1. 查找

   1. 根据hash定位槽

   2. 在槽中查找给定key（hash相等、key相等），找到直接返回，否则最后返回null

      1. 若槽节点key相等，返回槽节点

      2. 若槽节点为树节点，委托给树查找

      3. 遍历链表查找

           

2. 遍历

   从`index = 0, table[index]`开始，找到一个不为null的槽，遍历链表

   

3. 插入

   1. 如果table为空，或长度为0，初始化。（默认loadFactor为0.75，默认capacity为16（capacity是table的长度），threshold一般为capacity*loadFactor。）

   2. 通过hash定位槽，如果槽为空，构造新节点赋值给槽

   3. 若槽不为空，则在槽的链表或树中找到key相同的节点，替换节点值为新值；或是没有key相同的节点，就在树中或链表尾部加入新节点；若链表加入新节点后长度达到8（槽不算，aka槽下原有7个节点），则进行红黑树转化

   4. 如果是新加入节点，modCount、元素个数size自增1，如果元素个数超过阈值，则进行扩容

        

4. 扩容

   1. 计算新容量newCap和新阈值newThr（ps: 当容量已到最大值时，不再扩容；2倍扩容；）

   2. 创建新的数组，赋值给table

   3. 将键值对重新映射到新数组上

      1. 如果无链表，直接根据`hash&(newCap-1)`定位

      2. 如果是树节点，委托红黑树来拆分和重新映射

      3. 为链表，根据`hash&oldCap`的值分成0、1两组，映射到j和j+oldCap（0低位，1高位）（**链表顺序不变**）

           

 5. 删除

    1. 定位到槽

    2. 找到删除节点

    3. 删除节点，并修复链表或红黑树

         

6. 链表树化

   1. 链表树化有两个条件，不满足采用扩容，满足再扩容
   2. 树化时，将Node节点替换为TreeNode，保留next信息
   3. 替换后，再从head开始，进行红黑树化（标记红黑节点、父子节点，如果root节点不是first节点，再修正next和prev？）【链表转成红黑树后，原链表的顺序仍然会被引用仍被保留了（红黑树的根节点会被移动到链表的第一位）】

   在扩容过程中，树化要满足两个条件：

   1. 链表长度大于等于 TREEIFY_THRESHOLD 8 

   2. 桶数组容量大于等于 MIN_TREEIFY_CAPACITY 64

        

7. 红黑树拆分（扩容时候）

   红黑树中保留了next引用，拆分原理和链表相似

   1. 根据hash拆分成两组（这时候会生成新的next关系）

   2. 各组内根据情况，链化或者重新红黑树化

        

8. 红黑树链化

   将TreeNode替换为Node



###### ConcurrentHashMap概述

相比较HashMap，主要是增加了写操作时候的同步处理。扩容迁移时，多个线程帮助迁移。



1. 为什么要用synchronized代替ReentrantLock？

   1. 优化后的synchronized性能与ReentrantLock差不多，基于JVM也保证synchronized在各平台上的使用一致。

   2. 锁粒度降低了；在大量数据操作下，基于api的ReentrantLock会有更大的内存开销。

        

2. sizeCtl

   1. 默认为0

   2. 当table为null时，持有一个initial table size用于初始化

   3. 当sizeCtl<0时

      1. -1表示正在初始化

      2. 非-1的负数

         ```java
         （sizeCtl的低16位-1）表示有多少个线程参与扩容迁移
          sizeCtl的高16位
         -(1 + the number of active resizing threads)
         ```

3. sizeCtl>0时，`(n << 1) - (n >>> 1) ` = 0.75n （表示阈值，超过阈值需要扩容）
           

4. 插入

   1. 计算hash

   2. 循环执行

      1. 如果数组为空，初始化initTable
         2. 如果hash定位到的槽为空，CAS替换为新节点，退出循环
         3. 如果槽不为空，节点hash为-1，说明正在迁移，helpTransfer
         4. 槽不为空，且不在迁移，那么，对头节点加锁，链表或红黑树形式插入或更新节点

   3. addCount

        

5. 迁移

   transfer的第二个参数为空的时候，触发扩容，创建nextTable，在addCount和tryPresize中有这样的调用。

   addCount是size不精确情况下，可能触发扩容；tryPresize是已知精确size的情况下做扩容。

   1. 计算步长stride
      2. 如果nextTab未创建，则创建之，并赋给nextTable
      3. 循环迁移
         1. 分配迁移区间i`和`bound`（`i`从前往后，`bound = i - stride + 1`，总之就是stride）
         2. 如果区间已达边界，将sc减1，表示本线程退出迁移。如果是最后一个迁移线程，标记finish和advance为true，进入下一循环recheck；非最后线程，直接退出方法。
         3. 如果finish为true，`table = nextTab; sizeCtl = (n << 1) - (n >>> 1);`，退出
         4. 若未达边界，且槽为空，CAS槽为fwd，进入下一循环
         5. 槽不为空，且槽已经是fwd，进入下一循环
         6. 最后一种情形，进行迁移
            1. 为链表，根据节点hash二进制第k位为0或1分成两组（n=2^k），1连接到高位槽上
            2. 为红黑树，分组同链表，分好的组根据节点个数判断是否链化或新生成红黑树



###### HashMap检测到hash冲突后，将元素插入在链表的末尾还是开头？

Java8是加载链表末尾

Java7是开头

头插法会改变链表中元素原本的顺序，在并发情况下**可能**会产生链表成环的问题。

Java7到Java8的改变[HashMap为何从头插入改为尾插入](https://blog.csdn.net/weixin_33919941/article/details/88031334)

java7的问题[老生常谈，HashMap的死循环](https://www.jianshu.com/p/1e9cf0ac07f4)

<img src="/github/northernw.github.io/image/IMG_2568.jpg" alt="IMG_2568" style="zoom:10%;" />



###### 1.8还采用了红黑树，讲讲红黑树的特性，为什么人家一定要用红黑树而不是AVL、B树之类的？

插入、删除、查找的最坏时间复杂度都为 O(logn)。

红黑树特性：

1. 每个节点要么是黑色，要么是红色
2. 根节点是黑色的
3. 每个叶节点是黑色的（Java实现中，叶子节点是null，遍历时看不到黑色的叶子节点，反而每个叶子节点是红色的）
4. 如果一个节点是红色的，那么它的两个子节点是黑色的（意味着可以有连续的黑色节点，但不能有连续的红色节点。若给定N个黑色节点，最短路径情况是连续N个黑色，树高为N-1；最长路径情况是红黑相间，树高为2N-2）
5. 对任一节点，从节点到它每个叶子节点的路径包含相同数量的黑色节点（最主要特性，插入、删除要调整以遵守这个规则）

[面试旧敌之红黑树（直白介绍深入理解）](https://juejin.im/entry/58371f13a22b9d006882902d)



为什么用红黑树？

红黑树的统计性能（理解为增删查平均性能）优于AVL树。

AVL：名字来源发明者G. M. Adelson-Velsky和E. M. Landis。本质是平衡二叉搜索树（查找树），任何节点的左右子树高度差不超过1，是高度平衡的二叉查找树。

B树：[重温数据结构：理解 B 树、B+ 树特点及使用场景](https://juejin.im/entry/5b0cb64e518825157476b4a9) 平衡二叉树节点最多有两个子树，而 B 树每个节点可以有多个子树，**M 阶 B 树表示该树每个节点最多有 M 个子树**



AVL树高度平衡，查找效率高，但维护这个平衡的成本比较大，插入、删除要做的调整比较耗时。

红黑树的插入、删除、查找各种操作的性能比较平衡。

B树和B+树多用于数据存储在磁盘上的场景，比较矮胖，一次读取较多数据，减少IO。节点内是有序列表。列表的插入、删除成本比较高，如果是链表形式，则查找效率比较低（不能用二分查找提高查询效率）。

【自己的理解：B树节点内是有序列表，通过二分查找提高效率】

为什么STL和linux都使用红黑树作为平衡树的实现？ - Acjx的回答 - 知乎 https://www.zhihu.com/question/20545708/answer/58717264



###### 谈谈Java容器ArrayList、LinkedList、HashMap、HashSet的理解，以及应用场景

|                | ArrayList         | LinkedList        | HashMap     | HashSet           |
| -------------- | ----------------- | ----------------- | ----------- | ----------------- |
| 数据结构       | （可变）数组      | （双向）链表      | 数组+红黑树 | 底层实现是HashMap |
| 插入时间复杂度 | o(n)              | o(1)              |             |                   |
| 删除时间复杂度 | o(n)              | o(1)              |             |                   |
| 访问时间复杂度 | o(1) 支持随机访问 | o(n) 不支持随机.. |             |                   |
| 应用场景       | 经常访问          | 经常修改          | 映射..？    | 去重              |

###### sortset底层，原理，怎么保证有序

TreeSet具体实现是TreeMap，底层是红黑树

containsKey、get、put、remove 时间复杂度log(n)

<img src="/github/northernw.github.io/image/image-20200714155730096.png" alt="image-20200714155730096" style="zoom: 33%;" />

红黑树

通过对任何一条（根到叶子的）路径上的各个节点的着色方式的限制，确保没有一条路径会比其他路径长出2倍，因而近乎是平衡的

性质：

1. 每个节点是红色的，或是黑色的
2. 根节点是黑色的
3. 每个叶子节点（Nil）是黑色的
4. 如果一个节点是红色的，则它的两个子节点是黑色的
5. 对每个节点，从该节点到其子孙节点的所有路径上包含相同个数的黑色节点。（红节点不能有红孩子）（从该节点出发的所有下降路径，有相同的黑节点个数）



黑高度：从一个节点到达一个叶子节点的任意一条路径上黑色节点的个数

红黑树的黑高度定义为根节点的黑高度



###### 优先级队列的底层原理？

堆，默认是小顶堆

入队

```java
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        siftUp(i, e);
        size = i + 1;
        return true;
    }
    private static <T> void siftUpComparable(int k, T x, Object[] es) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        while (k > 0) {
          // 如果父节点比自己大
            int parent = (k - 1) >>> 1;
            Object e = es[parent];
            if (key.compareTo((T) e) >= 0)
                break;
            es[k] = e;
            k = parent;
        }
        es[k] = key;
    }
```

出队

```java
    public E poll() {
        final Object[] es;
        final E result;

        if ((result = (E) ((es = queue)[0])) != null) {
            modCount++;
            final int n;
            final E x = (E) es[(n = --size)];
            es[n] = null;
            if (n > 0) {
                final Comparator<? super E> cmp;
                if ((cmp = comparator) == null)
                    siftDownComparable(0, x, es, n);
                else
                    siftDownUsingComparator(0, x, es, n, cmp);
            }
        }
        return result;
    }
    private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
        // assert n > 0;
        Comparable<? super T> key = (Comparable<? super T>)x;
        int half = n >>> 1;           // loop while a non-leaf
        while (k < half) {
          // 从孩子中选一个小的
            int child = (k << 1) + 1; // assume left child is least
            Object c = es[child];
            int right = child + 1;
            if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
                c = es[child = right];
            if (key.compareTo((T) c) <= 0)
                break;
            es[k] = c;
            k = child;
        }
        es[k] = key;
    }
```



###### DelayQueue

https://www.cnblogs.com/jobs/archive/2007/04/27/730255.html

DelayQueue = BlockingQueue + PriorityQueue + Delayed



## Java并发

###### 线程池的工作原理，几个重要参数，然后给了具体几个参数分析线程池会怎么做，最后问阻塞队列的作用是什么？

线程池解决两个问题：

1. 由于减少了每个任务的调度开销，通常在执行大量异步任务时提供优秀的性能。
2. 提供了管理、调控资源的方式



Executors工厂方法：

1. newFixedThreadPool 固定size的线程池。为了满足资源管理的需求，需要限制当前线程数量的场景。适用于负载比较重的服务器。
   1. corePoolSize == maximumPoolSize
   2. keepAliveTimes = 0
   3. LinkedBlockingQueue  队列大小Integer.MAX_VALUE，等价于无界
   4. 当线程池中线程数达到corePoolSize后，新任务将在队列中等待
   5. 由于使用无界队列，运行中的线程池不会拒绝任务
2. newSingleThreadExecotor 单个线程的线程池。需要保证顺序执行任务的场景，并且在任意时间点不会有多个线程是活动的。
   1. corePoolSize = maximumPoolSize  = 1
   2. keepAliveTimes = 0
   3. LinkedBlockingQueue
   4. 如果当前线程池无线程，就创建一个线程来运行任务
   5. 当线程数达到1后，新的任务都加入到队列中
3. newCachedThreadPool 大小无界的线程池（自动资源回收？），适用于有很多短期异步执行任务的小程序，或者是负载比较轻的服务器。
   1. corePoolSize = 0, maximumPoolSize = Integer.MAX_VALUE
   2. keepAliveTimes = 60s
   3. SynchronousQueue 是一个没有容量的阻塞队列，一个插入操作必须等待另一个线程对应的移除操作
   4. 提交任务时如果有空闲线程，就空闲线程取到这个任务执行；否则创建一个线程来执行任务
   5. 适用于将主线程的任务传递给空闲线程执行



重要参数：

1. core and maximum pool sizes 
   1. corePoolSize 核心最大线程：新任务加入时，如果运行线程个数小于核心线程数，即使有其他工作线程是空闲的，也会创建新线程 -- 线程池预热
   2. maximumPoolSize 线程池最大线程：阻塞队列满时，如果运行线程数小于maximumPoolSize，才可创建新线程运行任务
   3. corePoolSize=maximumPoolSize时，等价于newFixedThreadPool
   4. maximumPoolSize=本质上无限的数（比如Integer.MAX_VALUE），等价于newCachedThreadPool ？
   5. 一般只在构造时设置这两个参数，但也可以通过两个set方法改变
   6. **这两个参数会自动调整么？**
2. On-demand construction
   1. 默认情况下，只有任务提交时才会创建线程（包括核心线程）
   2. 也可以通过prestartCoreThread或者prestartAllCoreTheads来预先创建线程。比如构建了一个阻塞队列不为空的线程池时，会想要这么做（预先创建线程）。
3. Creating new threads
   1. 默认使用defaultThreadFactory来创建线程，相同的线程组ThreadGroup、优先级priority和非守护线程状态non-daemon status.
   2. 也可以使用自定义的threadFactory，自定义线程名称、线程组、优先级等。
   3. threadFactory创建线程失败的什么东西没看懂
4. Keep-alive times
   1. keepAliveTime 如果线程数多于核心线程数，超过这个时间的空闲线程将会被停掉（指销毁掉？）
5. queuing
   1. 入队规则
6. rejected tasks 四个拒绝策略 RejectedExecutionHandler
   1. ThreadPoolExecutor.AbortPolicy 抛出RejectedExecutionException
   2. CallerRunsPolicy 调用者自身来执行
   3. DiscardPolicy 丢弃任务，任务不会被执行
   4. DiscardOldestPolicy work queue的首个任务将会被丢弃，重试添加当前任务（可能再次失败，自旋执行）
7. hook methods
   1. beforeExecute afterExecute 可用来设置运行环境，重新初始化本地线程，获取统计数据，添加日志。
   2. terminated executor终止时提供的钩子方法
8. queue maintenance getQueue可用于监控和调试当前work queue，其他用途不建议。remove和purge可用于大量任务取消时候的存储清理。
9. reclamation （清除？）一个在程序中无引用、并且无剩余线程的线程池，即使无显式shutdown关闭，也可以被清除回收。可以通过这些方式设置线程池的线程在无使用时（最终）销毁：设置keep-alivet times；使用小的核心线程数比如0，或者设置allowCoreThreadTimeOut。



ScheduledThreadPoolExcutor

延迟运行命令，或周期执行命令



LinkedBlockingQueue和DelayQueue的实现原理

1. LinkedBlockingQueue 就是生产者消费者的实现
   1. 应用了ReentrantLock（putLock & tackLock）和lock的Condition（notEmpty & notFull）
2. DelayQueue
   1. 应用了PriorityQueue，时间小的在队头
   2. ReentrantLock（lock）和Condition（available）



FutureTask是用AQS实现的 get=acquireShared，run/cancel后=release



###### 谈谈Java线程的基本状态，其中的wait() sleep() yield()方法的区别。

[线程的基本状态](https://juejin.im/post/5ae6cf7a518825670960fcc2)

新建、运行（运行中、就绪）、等待、超时等待、阻塞、终止

![image-20200513105703453](/github/northernw.github.io/image/image-20200513105703453.png)



wait()

Object的方法，在某个对象上等待，等待这个对象将它唤醒，释放锁。运行->等待/超时等待

sleep()

Thread的静态方法，当前线程睡眠，不释放锁。运行->超时等待

yield()

Thread的方法，让出当前cpu。还是运行这个大状态，从运行中变成就绪状态。



###### 简单谈谈JVM内存模型，以及volatile关键字

**运行时数据区域**包括堆、方法区（包括运行时常量池）、Java虚拟机栈、本地方法栈、程序计数器、直接内存。

1. 堆：所有对象在这里分配内存【所有线程共享】
2. 方法区：存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等信息【所有线程共享】
3. Java虚拟机栈：生命周期与线程相同，描述的是Java方法执行时候的内存模型，每个方法被执行的时候都会创建一个栈帧，存储局部变量表、操作数栈、常量池引用（动态链接）、方法出口等信息。【线程私有】
4. 本地方法栈：与虚拟机栈类似，只不过方法是本地方法【线程私有】
5. 程序计数器：记录正在执行的虚拟机字节码指令的地址（如果是本地方法则为空）【线程私有】
6. 直接内存：JDK1.4引入NIO，可以使用native函数库分配堆外内存，然后通过堆内的DirectByteBuffer作为这部分内存的引用、进行操作。可以提高性能，避免堆外内存和堆内内存的来回拷贝。



**Java内存模型 JMM**

**Java memory model**

用来屏蔽不同硬件和操作系统的内存访问差异，实现Java在各平台上一致的内存访问效果。

JMM规定，所有变量都存在主内存中（类似于操作系统的普通内存）；每个线程有自己的工作内存（=CPU的寄存器或高速缓存），保存了该线程使用的变量的主内存副本拷贝。线程只能操作工作内存。

存在缓存不一致问题。



<img src="/github/northernw.github.io/image/image-20200514161253557.png" alt="image-20200514161253557" style="zoom:30%;" />



**主内存与工作内存交互操作**

<img src="/github/northernw.github.io/image/image-20200514163326711.png" alt="image-20200514163326711" style="zoom:50%;" />

**内存模型三大特性**

1. 原子性：上述8个操作是原子的（double&long等64位变量的操作，JVM允许非原子），一系列操作合起来其实是非原子的

2. 可见性：指一个线程修改了共享变量的值，其他线程可以立即得知这个修改。JMM是通过变量修改后将新值同步到主内存（并使其他工作内存中的这个变量副本无效）、在变量读取前从主内存刷新变量值来实现的。

3. 有序性：从本线程来看，所有操作都是有序的；从线程外看，操作是无序的，因为发生了指令重排。JMM允许编译器和处理器进程指令重排，重排不会影响到单线程的执行结果，但会影响多线程的执行正确性。

   volatile关键字通过添加内存屏障的方式来禁止指令重排（重排序时不能把屏障后的指令重排到屏障前）



**先行发生原则**

1. 单一线程原则：在一个线程内，在程序前面的操作先行发生于后面的操作。
2. 管程锁定原则：一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。
3. volatile变量规则：对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。
4. 线程启动规则：Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。
5. 线程加入规则：Thread 对象的结束先行发生于 join() 方法返回。
6. 线程中断规则：对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生。
7. 对象终结规则：一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。
8. 传递性：如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。



[volatile关键字](https://juejin.im/post/5a2b53b7f265da432a7b821c#heading-0)

**volatile关键字**

1. 保证了不同线程对该变量操作的内存可见性
2. 禁止指令重排序，保证（volatile读写）有序性



###### volatile的底层如何实现，怎么就能保住可见性了？

具体在👆

在缓存行和主内存之间，利用的是缓存一致性协议。

在写入缓存和缓存行之间，利用的是内存屏障。

从规范上讲是内存屏障，x86实现上是lock前缀指令，既有原子性的效果，也有StoreLoad内存屏障的效果。

内存屏障的保守插入方式，为了使写操作一定刷新到缓存行，（因为缓存一致性和禁止重排序）读操作一定读到最新值：

1. 在每个volatile读后面，插入LoadLoad和LoadStore
2. 在每个volatile写前面插入StoreStore，写后面插入StoreLoad

```java
public class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;
        // StoreStore 确保a的值已写入
        flag = true;
        // StoreLoad 确保flag的值在后面的Load之前已写入
    }

    public void reader() {
        if (flag) {
            // volatile读后 后面的写和读不能重排序到读flag前，确保是基于最新flag值做操作
            // LoadStore
            // LoadLoad
            int i = a;
            System.out.println(i);
        }
    }
}
```



###### 线程之间的交互方式有哪些？有没有线程交互的封装类？

1. 线程之间的协作

   1. join() 在线程中调用另一个线程的join()方法，会将本线程挂起，直到目标线程结束
   2. wait() notify() notifyAll()
      1. 调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。
      2. 属于Object（不是Thread）
   3. await() signal() signalAll()
      1. java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用Condition的 signal() 或 signalAll() 方法唤醒等待的线程。

2. 线程交互的封装类

   1. CountDownLatch

      1. 用来控制一个线程等待多个线程。

      2. 维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方 法而在等待的线程就会被唤醒。

      3. ```java
         public class CountdownLatchExample {
            public static void main(String[] args) throws InterruptedException {
                final int totalThread = 10;
                CountDownLatch countDownLatch = new CountDownLatch(totalThread);
                ExecutorService executorService = Executors.newCachedThreadPool();
                for (int i = 0; i < totalThread; i++) {
                    executorService.execute(() -> {
                        System.out.print("run..");
                        countDownLatch.countDown();
         }); }
                 countDownLatch.await();
                 System.out.println("end");
                 executorService.shutdown();
         } }
         
         run..run..run..run..run..run..run..run..run..run..end
         ```

         等待所有run结束

   2. CyclicBarrier

      1. 用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

      2. 和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

      3. CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

      4. ```java
         public class CyclicBarrierExample {
             public static void main(String[] args) {
                 final int totalThread = 10;
                 CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
                 ExecutorService executorService = Executors.newCachedThreadPool();
                 for (int i = 0; i < totalThread; i++) {
                     executorService.execute(() -> {
                         System.out.print("before..");
                         try {
                             cyclicBarrier.await();
                         } catch (InterruptedException | BrokenBarrierException e) {
                             e.printStackTrace();
                         }
                         System.out.print("after..");
                     });
         }
                 executorService.shutdown();
             }
         }
         
         before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..after..
         ```

         等待所有before结束

         

   3. Semaphore

      1. Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

      2. ```java
         public class SemaphoreExample {
             public static void main(String[] args) {
                 final int clientCount = 3;
                 final int totalRequestCount = 10;
                 Semaphore semaphore = new Semaphore(clientCount);
                 ExecutorService executorService = Executors.newCachedThreadPool();
                 for (int i = 0; i < totalRequestCount; i++) {
                     executorService.execute(()->{
                         try {
                             semaphore.acquire();
                             System.out.print(semaphore.availablePermits() + " ");
                         } catch (InterruptedException e) {
                             e.printStackTrace();
                         } finally {
                             semaphore.release();
                         }
         }); }
                 executorService.shutdown();
             }
         }
         1 0 1 1 1 2 2 2 0 1 
         ```

         有限个资源



###### Java的锁机制 -- 内容巨多

[Java锁机制](https://juejin.im/post/5e0c5eba6fb9a047ef326a0b)  [AQS机制](https://juejin.im/post/5e16effef265da3e491a3f5e)

1. 背景知识

   1. 指令流水线：现代处理器的体系结构中，采用流水线的方式对指令进行处理。每个指令的工作可分为5个阶段：取指令、指令译码、执行指令、访存取数和结果写回。
   2. CPU多级缓存：计算机系统中，存在CPU高速缓存，用于减少处理器访问内存所需平均时间。当处理器发出内存访问请求时，会先查看缓存中是否有请求数据，若命中则直接返回该数据；若不存在，则先从内存中将数据载入缓存，再将其返回处理器。

2. 问题引入

   1. 原子性：即一个操作或者多个操作 要么全部执行**并且执行的过程不会被任何因素打断**，要么就都不执行。（比如i++，如果对实例变量i的操作不做额外的控制，那么多个线程同时调用，就会出现覆盖现象，丢失部分更新。） -- 因为指令流水线
   2. **可见性**：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值（存在可见性问题的根本原因是由于缓存的存在）-- 因为存在缓存
   3. **顺序性**：即程序执行的顺序按照代码的先后顺序执行 -- 因为存在指令重排

3. JMM内存模型

   主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。这里的变量指共享变量（存在竞争问题的变量），如实例字段、静态字段、数组对象元素等。不包括线程私有的局部变量、方法参数等。

   1. 内存划分：分为主内存和工作内存，【每个线程都有自己的工作内存，它们共享主内存。】【线程对共享变量的所有读写操作都在自己的工作内存中进行，不能直接读写主内存中的变量。】【不同线程间也无法直接访问对方工作内存中的变量，线程间变量值的传递必须通过主内存完成。】

      - 主内存（Main Memory）存储所有共享变量的值。

      - 工作内存（Working Memory）存储该线程使用到的共享变量在主内存的的值的副本拷贝。

   2. 内存间交互规则【一个变量如何从主内存拷贝到工作内存，如何从工作内存同步到主内存中】

      ![image-20200603190808840](/github/northernw.github.io/image/image-20200603190808840.png)

      **8种原子操作**

      - lock: 将一个变量标识为被一个线程独占状态
      - unclock: 将一个变量从独占状态释放出来，释放后的变量才可以被其他线程锁定
      - read: 将一个变量的值从主内存传输到工作内存中，以便随后的load操作
      - load: 把read操作从主内存中得到的变量值放入工作内存的变量的副本中
      - use: 把工作内存中的一个变量的值传给执行引擎，每当虚拟机遇到一个使用到变量的指令时都会使用该指令
      - assign: 把一个从执行引擎接收到的值赋给工作内存中的变量，每当虚拟机遇到一个给变量赋值的指令时，都要使用该操作
      - store: 把工作内存中的一个变量的值传递给主内存，以便随后的write操作
      - write: 把store操作从工作内存中得到的变量的值写到主内存中的变量

      **原子操作的使用规则**

      - read、load、use必须成对顺序出现，但不要求连续出现。assign、store、write同之；

      - 变量诞生和初始化：变量只能从主内存“诞生”，且须先初始化后才能使用，即在use/store前须先load/assign；

      - lock一个变量后会清空工作内存中该变量的值，使用前须先初始化；unlock前须将变量同步回主内存；

      - 一个变量同一时刻只能被一线程lock，lock几次就须unlock几次；未被lock的变量不允许被执行unlock，一个线程不能去unlock其他线程lock的变量。

      对于double和long，虽然内存模型允许对非volatile修饰的64位数据的读写操作分为两次32位操作来进行，但商用虚拟机几乎把64位数据的读写实现为了原子操作，可以忽略这个问题。

   3. 先行发生原则

      【Java内存模型具备一些先天的“有序性”，即不需要通过任何同步手段（volatile、synchronized等）就能够得到保证的有序性，这个通常也称为happens-before原则。】

      如果两个操作的执行次序不符合先行原则且无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

      1. 程序次序规则（Program Order Rule）：一个线程内，逻辑上书写在前面的操作先行发生于书写在后面的操作。
      2. 监视器锁规则（Monitor Lock Rule）：一个unLock操作先行发生于后面对同一个锁的lock操作。“后面”指时间上的先后顺序。
      3. **volatile变量规则**（Volatile Variable Rule）：对一个volatile变量的写操作先行发生于后面对这个变量的读操作。“后面”指时间上的先后顺序。
      4. 传递规则（Transitivity）：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C。
      5. 线程启动规则（Thread Start Rule）：Thread对象的start()方法先行发生于此线程的每个一个动作。
      6. 线程中断规则（Thread Interruption Rule）：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生（通过Thread.interrupted()检测）。
      7. 线程终止规则（Thread Termination Rule）：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行。
      8. 对象终结规则（Finaizer Rule）：一个对象的初始化完成（构造函数执行结束）先行发生于他的finalize()方法的开始。

4. 问题解决

   1. 原子性

      1. 由JMM保证的原子性变量操作
      2. 基本数据类型的读写（工作内存）是原子的
      3. JMM的lock和unlock指令可以实现更大范围的原子性保证，虚拟机提供synchronized关键字和Lock锁来保证原子性。

   2. 可见性

      1. **volatile关键字**修饰的变量，被线程修改后会立即同步回主内存，其他线程要读取这个变量会从主内存刷新值到工作内存。（因为缓存一致性协议会让其他工作内存中的该变量拷贝无效，必须得从主内存再读取）即read、load、use三者连续顺序执行，assign、store、write连续顺序执行。

   3. **synchronized/Lock** ~~由lock和unlock的使用规则保证【这里有疑问啊，synchronized有lock和unlock，但是Lock没有吧...Lock怎么保证可见性？还是说Lock保证不了可见性。可见性只能由volatile保证？--参见ConcurrentHashMap，有synchronized，还配合volatile使用---ConcurrentHashMap有些是不加锁的操作，比如get，所以还是用volatile保证可见性。synchronized 锁的是某个node节点，对这个node节点的】~~

      1. synchronized有语义规定，说是通过内存屏障实现的

         线程解锁前，必须把共享变量的最新值刷新到主内存中
         线程加锁前，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值

         2. Lock用了cas，有`lock cmpxchg`，lock前缀指令保证了可见性，同时有内存屏障的作用

         **同时，这俩还能保证临界区操作的所有变量的可见性**因为内存屏障

         > LOCK前缀的指令具有如下效果：
         >
         > - 把写缓冲区中所有的数据刷新到内存中
         >
         > 注意，是所有的数据，可不仅仅是对state的修改

         > [ReentrantLock对可见性的支持](https://www.zhihu.com/question/41016480/answer/130906913)
         >
         > All threads will see the most recent write to a volatile field, along with any writes which preceded that volatile read/write. Reentrantlock的lock和unlock方法实际上会cas一个state的变量，state是volatile的，因此夹在两次state之间的操作都能保证可见性。这应该算是happen before的传递性...

   4. 顺序性

      1. volatile 禁止指令重排序
      2. synchronized/Lock “一个变量在同一个时刻只允许一条线程对其执行lock操作” -- 感觉这个也没用，不然双重检查的单例怎么还用volatile关键字来防止重排序 -- 最多保证原子性，被加锁内容按照顺序被多个线程执行

5. 锁机制

   1. volatile：

      保证可见性和顺序性【实现方式：lock前缀指令+依赖MESI缓存一致性协议】

      1. volatile修饰的变量，在进行写操作的时候会多一行汇编代码，lock指令，做两件事：
         1. 将当前处理器缓存行的数据写回系统内存
         2. 引起其他处理器里缓存了该内存地址的数据无效。【实现缓存一致性协议，处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了（处理器发现自己缓存行对应的内存地址被修改，就会将自己的缓存设置成无效状态）】

   2. final：有两个重排序规则 -- 不甚了解

      1. **写final域的重排序规则**：在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
      2. 读final域的重排序规则：初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

   3. synchronized关键字

      1. 使用哪个对象的监视器：

         - 修饰对象方法时，使用当前对象的监视器
         - 修饰静态方法时，使用类类型（Class 的对象）监视器
         - 修饰代码块时，使用括号中的对象的监视器
           - 必须为 Object 类或其子类的对象

      2. 无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁

         简单理解，只有一个线程CAS时，如果CAS成功，表示没有锁竞争，保持偏向锁状态，如果CAS失败，说明有竞争，（先撤销偏向锁，将对象头设置成无锁状态，并设置为不可偏向）升级为轻量级锁。

         几种锁的适用场景

         1. 偏向锁：锁不仅不存在线程竞争，而且总是由同一个线程多次获得，这时候偏向锁的代价最低。适用只有一个线程访问同步块的场景。（如果有别的线程来获取锁，发现）
         2. 轻量级锁：同步块执行时间非常快的，执行完就替换回mark word，别的线程要加锁也很快，CAS。（如果同步块执行很久，竞争线程自旋cas非常久，就很耗cpu，所以会升级到重量级锁，竞争线程阻塞挂起）
         3. 重量级锁：同步块执行时间比较长的，原因如2

         

         锁升级机制

         1. 偏向锁：线程检查锁对象的状态是否是可偏向的，是的话，检查mark word中的线程ID是不是自己，是的话进入代码块，不是的话，将线程ID cas进mark word。cas失败的话，说明之前是别的线程（假设A）取到的了，等待全局安全点，JVM暂停线程A，检查线程A的状态：如果A不在活动中，将锁对象的mark word中的线程ID置空，再cas成自己的线程ID；如果A在活动中（未退出代码块），升级为轻量级锁：JVM在线程A中分配锁记录，拷贝锁对象mark word，并将锁对象mark word指向这个锁记录；在线程B中分配锁记录，拷贝锁对象mark word，并持续自旋cas（如果自旋n次还失败，就要再次升级成重量级锁了..）...

         2. 轻量级锁：如果不止一个线程尝试获取锁，就会升级到轻量级锁。**通过自适应自旋CAS的方式获取锁。**如果获取失败，说明存在竞争，膨胀为重量级锁，线程阻塞。默认自旋10次。**将对象头中的Mark Word复制到栈帧（一块空间，称为锁记录）中，然后用CAS将对象头中的Mark Word替换为指向栈帧中锁记录的指针。**

         3. 重量级锁：通过系统的线程互斥锁来实现的，未获取到锁的线程会阻塞挂起

              

         大佬的图，来源见水印

         右下角的轻量级锁释放的补充说明：

         在某个线程A正持有轻量级锁的时候（还在代码块内运行，时间比较长），某个线程B自旋cas竞争锁（肯定是cas失败了）失败了，这时候就会升级成重量级锁了，mark word指向了互斥量的指针，这和线程A中锁记录的值不同，线程A后续释放锁就失败了（意识到已经升级成重量级锁，唤醒其他挂起的线程）

         ![img](/github/northernw.github.io/image/172a2f26935d33c8.png)

   4. AQS



###### 【内存屏障和"lock"前缀指令】理解

volatile通过编译器，既会增加"lock"前缀指令，也会加上内存屏障（mfence等）

内存屏障是抽象概念，各个硬件、处理器实现不同

lock前缀指令和mfence等是具体实现

mesi协议保证缓存和主存间的一致性

> 有了msei协议，为什么汇编层面还需要lock(volatile)来实现可见性？ - Rob Zhang的回答 - 知乎 https://www.zhihu.com/question/334662600/answer/747038084
>
> 内存屏障能保证从storebuffer到缓存再到主存的一致性，在多线程运行中可以作为mesi的补充（因为mesi管不到那么多），但内存屏障
>
> **lock前缀主要是为了提供原子操作，虽然它也包含了内存屏障功能**（强制将寄存器、缓存（、storebuffer/invalid queue或类似的东西）等强制同步到主存）



> 关于内存屏障的几个问题？ - cao的回答 - 知乎 https://www.zhihu.com/question/47990356/answer/108650501
>
> x86在Windows下的内存屏障是用lock前缀指令来达到效果的



**简单理解：**

**内存屏障保证了寄存器和缓存之间的一致性**

**lock前缀保证操作原子性**

**二者都能保证可见性**



###### x86架构的内存屏障

1. sfence: Store Barrier = StoreStore Barriers 写屏障

   所有sfence之前的store指令都在sfence之前被执行，并刷出到CPU的L1 Cache中；

   所有在sfence之后的store指令都在sfence之后执行，禁止重排序到sfence之前。

   所以，所有Store Barrier之前发生的内存更新都是可见的。

2. lfence: Load Barrier = LoadLoad Barriers 读屏障

   所有在lfence之后的load指令，都在lfence之后执行，并且一直等到load buffer被该CPU读完才能执行之后的load指令（即要刷新失效的缓存）。配合sfence，使所有sfence之前发生的内存更新，对lfence之后的load操作都可见。

3. mfence: Full Barrier = StoreLoad Barriers 全屏障

   综合了sfence和lfence的作用，强制所有在mfence之前的store/load指令都在mfence之前被执行，之后的store/load指令都在之后执行，禁止跨越mfence重排序。并且都刷新到缓存&重新载入无效缓存。



###### Mark Word 对象头【见JMM】 todo

主要有锁标志位，根据不同的锁状态其他位上存有不同的值，比如

1. 偏向锁：拥有锁的线程ID，偏向状态
2. 轻量级锁：拥有锁的锁记录地址
3. 重量级锁：监视器锁的地址



###### synchronized底层实现

加在方法上和加在同步代码块中编译后的区别、类锁、对象锁

编译时候加入监视器锁



```java
public class SyncTest {
    public void syncBlock() {
        synchronized (this) {
            System.out.println("hello block");
        }
    }

    public synchronized void syncMethod() {
        System.out.println("hello method");
    }
}
```



加在方法上：方法上有synchronized关键字，flags里有ACC_SYNCHRONIZED

https://blog.csdn.net/hosaos/java/article/details/100990954

ACC_SYNCHRONIZED是获取监视器锁的一种隐式实现(没有显示的调用monitorenter，monitorexit指令)

如果字节码方法区中的ACC_SYNCHRONIZED标志被设置，那么线程在执行方法前会先去获取对象的monitor对象，如果获取成功则执行方法代码，执行完毕后释放monitor对象

```shell
public synchronized void syncMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String hello method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 15: 0
        line 16: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lwyq/learning/quickstart/juc/SyncTest;
```

加在同步块上：monitorenter / monitorexit 关键字 

```shell
public void syncBlock();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3                  // String hello block
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 9: 0
        line 10: 4
        line 11: 12
        line 12: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  this   Lwyq/learning/quickstart/juc/SyncTest;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class wyq/learning/quickstart/juc/SyncTest, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
```

###### volatile在编译上的体现

```java
    public class VolatileTest {
        private volatile int i;
    
        public void plus() {
            i = 2;
        }
    }
```

字节码
网上查到的是变量上flags有ACC_VOLATILE标识，自己编译出来没看到...

```shell
      public void plus();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=2, locals=1, args_size=1
             0: aload_0
             1: iconst_2
             2: putfield      #2                  // Field i:I
             5: return
          LineNumberTable:
            line 11: 0
            line 12: 5
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0       6     0  this   Lwyq/learning/quickstart/juc/VolatileTest;
```

看文章说还是lock前缀指令
 http://gee.cs.oswego.edu/dl/jmm/cookbook.html  -- x86架构下，实现是lock前缀指令，支持"SSE2"扩展 (Pentium4 and later)的版本支持mfence指令（比lock前缀更推荐），cas的cmpxchg的实现需要lock前缀


​    

https://www.cnblogs.com/xrq730/p/7048693.html
    

1. 锁总线，其它CPU对内存的读写请求都会被阻塞，直到锁释放，不过实际后来的处理器都采用锁缓存替代锁总线，因为锁总线的开销比较大，锁总线期间其他CPU没法访问内存
2. lock后的写操作会回写已修改的数据，同时让其它CPU相关缓存行失效，从而重新从主存中加载最新的数据
3. 不是内存屏障却能完成类似内存屏障的功能，阻止屏障两边的指令重排序

整理一下最终的实现：

1. lock前缀指令会引起处理器缓存回写到内存
2. 一个处理器的缓存回写到内存会导致其他处理器的缓存无效，这是MESI实现的（缓存一致性协议）
3. 另外，lock前缀指令能完成内存屏障的功能，阻止屏障前后的指令重排序


​    

这篇文章https://juejin.im/post/5ea938426fb9a043856f2f6a提到，x86下使用`lock`来实现`StoreLoad`，并且只有 `StoreLoad` 有效果。x86 上怎么使用 Barrier 的说明可以在 openjdk 的代码中看到，在这里[src/hotspot/cpu/x86/assembler_x86.hpp](https://github.com/openjdk/jdk/blob/9a69bb807beb6693c68a7b11bee435c0bab7ceac/src/hotspot/cpu/x86/assembler_x86.hpp)。


​    




###### 3种重排序类型

1是编译器重排序，2和3是处理器重排序。会导致多线程程序出现内存可见性问题。

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

2. 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-LevelParallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

3. 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。



###### aqs，countDownLatch如何实现 todo



###### 计算密集型/IO密集型 任务 分别如何设置线程池的核心线程数和最大线程数，为什么这么设置

https://blog.csdn.net/weixin_40151613/java/article/details/81835974

计算密集型：

CPU使用率比较高，（也就是一些复杂运算，逻辑处理）

**线程数设置为CPU核数**



IO密集型：

cpu使用率较低，程序中会存在大量I/O操作占据时间，导致线程空余出来

**一般设置线程数为CPU核数的2倍**

最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目

线程等待时间越长，需要越多的线程



补充

1. 高并发、任务执行时间短的业务：线程池线程数可以设置为CPU核数+1，减少线程上下文的切换

2. 并发不高、任务执行时间长的业务：
   1. 假如是业务时间长集中在IO操作上，也就是IO密集型的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以适当加大线程池中的线程数目，让CPU处理更多的业务
   2. 假如是业务时间长集中在计算操作上，也就是计算密集型任务，和（1）一样，线程池中的线程数设置得少一些，减少线程上下文的切换

3. 并发高、业务执行时间长，解决这种类型任务的关键不在于线程池而在于整体架构的设计
   1. 数据能否做缓存
   2. 增加服务器
   3. 业务执行时间长的问题，也可能需要分析一下，看看能不能使用中间件（任务时间过长的可以考虑拆分逻辑放入队列等操作）对任务进行拆分和解耦。



###### 死锁

死锁定义：多个进程循环等待它方占有的资源而无限期地僵持下去的局面。

产生死锁的必要条件：

1. 互斥（mutualexclusion），一个资源每次只能被一个进程使用
2. 不可抢占（nopreemption），进程已获得的资源，在未使用完之前，不能强行剥夺
3. 占有并等待（hold andwait），一个进程因请求资源而阻塞时，对已获得的资源保持不放
4. 环形等待（circularwait），若干进程之间形成一种首尾相接的循环等待资源关系。

对待死锁的策略主要有：

1. 死锁预防：破坏导致死锁必要条件中的任意一个就可以预防死锁。例如，要求用户申请资源时一次性申请所需要的全部资源，这就破坏了保持和等待条件；将资源分层，得到上一层资源后，才能够申请下一层资源，它破坏了环路等待条件。预防通常会降低系统的效率。

2. 死锁避免：避免是指进程在每次申请资源时判断这些操作是否安全，例如，使用银行家算法。死锁避免算法的执行会增加系统的开销。

3. 死锁检测：死锁预防和避免都是事前措施，而死锁的检测则是判断系统是否处于死锁状态，如果是，则执行死锁解除策略。

4. 死锁解除：这是与死锁检测结合使用的，它使用的方式就是剥夺。即将某进程所拥有的资源强行收回，分配给其他的进程。





###### 避免死锁的几个常见方法

1. 避免一个线程同时获取多个锁

2. 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。

3. 尝试使用定时锁，使用`lock.tryLock(timeout)`来代替使用内部锁机制。

4. 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。

    



## Java虚拟机

**虚拟机的几大问题**

1. 运行时数据区域
2. 垃圾收集
   1. 对象可达
   2. 引用类型
   3. GC Roots
   4. 算法
   5. 收集器
3. 内存分配与回收策略（回收主要是老年代的触发条件）
4. 类加载机制



###### 新生代分为几个区？使用什么算法进行垃圾回收？为什么使用这个算法？

新生代有三个区，一个较大的Eden区，两个小的Survivor区。

使用复制算法。（也有标记过程，标记-复制）

一方面，针对算法本身，相对于标记-清除算法，不会有内存碎片的问题；相对于标记-整理算法，处理效率高很多（在整理时，还未进行对象清理，移动存活对象时需要将存活对象插入到待清理对象之前，有大量的移动操作，时间复杂度很高）。

复制算法主要问题在于内存利用率，而HotSpot的Eden和Survivor的默认比例是8:1，保证内存利用率达到了90%，所以影响也不是太大。

另一方面，新生代minor gc比较频繁，对gc效率有比较高的要求；对象生命周期比较短，小的survivor空间即可容纳大部分情况下的存活对象。



引申：jvm的几个知识点，算法，判断对象存活，GC roots有哪些，内存分配与回收策略，类加载机制



###### 垃圾收集算法【见Java虚拟机】

1. 标记-清除
2. 标记-整理
3. 复制
4. 分代收集
   1. 新生代：复制算法
   2. 老年代：标记-清除 or 标记整理



###### 垃圾收集器与内存分配策略【祥见JVM的几个大知识点】

**垃圾收集器**

<img src="/Users/wangyuanqing1/Library/Application Support/typora-user-images/image-20200515150418111.png" alt="image-20200515150418111" style="zoom:50%;" />

**内存分配策略**

Minor GC 和 Full GC

1. Minor GC:回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比 较快。
2. Full GC:回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多。



分配策略

1. 对象优先在 Eden 分配

   大多数情况下，对象在新生代 Eden 上分配，当 Eden 空间不够时，发起 Minor GC。 

2. 大对象直接进入老年代

   大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

   经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。

   -XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 和 Survivor 之间的大量内存复制。

3. 长期存活的对象进入老年代

   为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁， 增加到一定年龄则移动到老年代中。

   -XX:MaxTenuringThreshold 用来定义年龄的阈值。

4. 动态对象年龄判定

   虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄 所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

5. 空间分配担保

   在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。

   如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，如果允许那么就会继续检查老年代 最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC;如果小 于，或者 HandlePromotionFailure 的值不允许冒险，那么就要进行一次 Full GC。

   

**Full GC 的触发条件**

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件:

1. 调用 System.gc()
   只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。 

2. 老年代空间不足

   老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

   为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数 调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对 象进入老年代的年龄，让对象在新生代多存活一段时间。

3. 空间分配担保失败
   使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

4. JDK 1.7 及以前的永久代空间不足

   在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静 态变量等数据。

   当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也 会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

   为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

5. Concurrent Mode Failure

   执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时 性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。



###### ClassLoader原理和应用

1. ClassLoader的作用

   1. 加载class字节码文件到jvm
   2. 确认每个类应由那个类加载器加载，这也影响到两个类是否相等的判断，影响的方法有equals()、isAssignableFrom()、isInstance()以及instanceof关键字

2. 加载的类存放在哪里？

   jdk8之前在方法区，8之后在元数据区。

3. 什么时候触发类加载？

   1. 隐式加载
      1. 遇到new、getstatic、putstatic、invokestatic4条字节码指令时
      2. 对类进行反射调用时
      3. 当初始化一个类时，如果父类还没初始化，优先加载父类并初始化
      4. 虚拟机启动时，需指定一个包含main函数的主类，优先加载并初始化这个主类
   2. 显式加载
      1. 通过ClassLoader的loadClass方法
      2. 通过Class.forName
      3. 通过ClassLoader的findClass方法

4. 有哪些类加载器ClassLoader？

   1. Bootstrap ClassLoader：加载JVM自身工作需要的类，由JVM自己实现。加载JAVA_HOME/jre/lib下的文件
   2. ExtClassLoader：是JVM的一部分，由`sun.misc.Launcher$ExtClassLoader`实现，会加载JAVA_HOME/jre/lib/ext下的文件，或由`System.getProperty("java.ext.dirs")`指定的目录下的文件
   3. AppClassLoader：应用类加载器，由`sun.misn.Launcher$AppClassLoader`实现，加载`System.getProperty("java.class.path")`目录下的文件，也就是classpath路径。

5. 双亲委派模型

   1. 原理：当一个类加载器收到类加载请求时，如果存在父类加载器，会先由父类加载器进行加载，当父类加载器找不到这个类时（根据类的全限定名称。找不到是由于，每个类有自己的加载路径。），当前类加载器才会尝试自己去加载。

   2. 为什么使用双亲委派模型？它可以解决什么问题？

      双亲委派模型能够保证类在内存中的唯一性。

      假如没有双亲委派模型，用户自己写了个全限定名为`java.lang.Object`的类，并用自己的类加载器去加载，同时BootstrapClassLoader加载了rt.jar包中的jdk本身的`java.lang.Object`，这样内存中就存在两份Object类了，会出现很多问题，例如根据全限定名无法定位到具体的类。



###### 高吞吐量的话用哪种gc算法

高吞吐量，如果指cpu多用于用户程序，需要停顿时间比较短的收集器，新生代在服务端一般用Parallel Scavenge，算法也是复制算法。

复制算法的性能比较高。



###### jvm参数调优详细过程，到为什么这么设置，好处，一些gc场景，如何去分析gc日志

jvm调优的基本原则：

1. 大多数Java应用不需要进行JVM优化
2. 大多数导致GC频繁、内存使用率高的问题的原因是代码层面的问题（代码层面）
3. 上线前应考虑将JVM参数设置最优
4. 减少创建对象的数量（代码层面）
5. 较少使用全局变量和大对象（代码层面）
6. 优先架构调优和代码调优，JVM优化是不得已的手段，或者说是发现问题
7. 分析gc情况优化代码比优化JVM参数更好（代码层面）



https://juejin.im/post/5dea4cb46fb9a01626644c36

新生代配置原则：

1.追求响应时间优先  这种需求下，新生代尽可能设置大一些，并通过实际情况调整新生代大小，直至接近系统的最小响应时间。因为新生代比较大，发生垃圾回收的频率会比较低，响应时间快速。

2.追求吞吐量优先  吞吐量优先的应用，在新生代中的大部分对象都会被回收，所以，新生代尽可能设置大。此时不追求响应时间，垃圾回收可以并行进行。

3.避免设置过小新生代   设置过小，YGC会很频繁，同时，很可能导致对象直接进入老年代中，老年代空间不足发生FullGC。

老年代配置原则：

1.追求响应时间优先   这种情况下，可以使用CMS收集器，以获取最短回收停顿时间，但是其内存分配需要注意，如果设置小了会造成回收频繁并且碎片变多；如果设置大了，回收的时间会很长。所以，最优的方案是根据GClog分析垃圾回收信息，调整内存大小。

2.追求吞吐量优先   吞吐量优先通常需要分配一个大新生代、小老年代，将短期存活的对象在新生代回收掉。



###### JVM性能调优的监控工具了解那些？

jps jstack jmap

**jps [option]**

输出Java进程信息

```shell
jps -ml
111957 org.apache.catalina.startup.Bootstrap -config /export/Domains/testenv.jd.local/server1/conf/server.xml start
136044 sun.tools.jps.Jps -ml
```

**jstack [option] pid**

输出某个进行内的线程栈信息

```shell
jstack 111957 | grep 1b6d0
"System_Clock" #307 daemon prio=5 os_prio=0 tid=0x00007f71b53f3800 nid=0x1b6d0 runnabl
e [0x00007f72606d9000]
```

```shell
-l long listings，会打印出额外的锁信息，在发生死锁时可以用<strong>jstack -l pid</strong>来观察锁持有情况  
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）  
```

**jmap [option] pid**

输出某个进程内的堆信息：JVM版本、使用的GC算法、堆配置、堆内存使用情况

```shell
jmap -heap 111957
Attaching to process ID 111957, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.20-b23

using thread-local object allocation.
Parallel GC with 43 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2147483648 (2048.0MB)
   NewSize                  = 357564416 (341.0MB)
   MaxNewSize               = 715653120 (682.5MB)
   OldSize                  = 716177408 (683.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 353370112 (337.0MB)
   used     = 28186432 (26.88067626953125MB)
   free     = 325183680 (310.11932373046875MB)
   7.976461801047849% used
From Space:
   capacity = 2097152 (2.0MB)
   used     = 1736768 (1.65631103515625MB)
   free     = 360384 (0.34368896484375MB)
   82.8155517578125% used
To Space:
   capacity = 2097152 (2.0MB)
   used     = 0 (0.0MB)
   free     = 2097152 (2.0MB)
   0.0% used
PS Old Generation
   capacity = 869793792 (829.5MB)
   used     = 160875768 (153.42308807373047MB)
   free     = 708918024 (676.0769119262695MB)
   18.495851485681793% used

36932 interned Strings occupying 3347024 bytes.
```

输出堆内存中对象个数、大小统计直方图

```shell
jmap -histo:live 111957 | less
```

![image-20200513150745023](/github/northernw.github.io/image/image-20200513150745023.png)

```shell
B  byte  
C  char  
D  double  
F  float  
I  int  
J  long  
Z  boolean  
[  数组，如[I表示int[]  
[L+类名 其他对象
```

dump出堆信息，再使用jhat或其他工具分析

```shell
jmap -dump:format=b,file=dump.dat 111957
jhat -port 8888 dump.dat
# 浏览器输入 ip:port可访问
```

**jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]** 

jvm统计信息

vmid是Java虚拟机ID，在Linux/Unix系统上一般就是进程ID。interval是采样时间间隔。count是采样数目。

```shell
jstat -gc 111957 250 6 # gc信息
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCS
C   CCSU   YGC     YGCT    FGC    FGCT     GCT
2048.0 2048.0  0.0    0.0   345088.0 148448.5 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
2048.0 2048.0  0.0    0.0   345088.0 148457.6 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
2048.0 2048.0  0.0    0.0   345088.0 148457.6 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
2048.0 2048.0  0.0    0.0   345088.0 150425.8 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
2048.0 2048.0  0.0    0.0   345088.0 150425.8 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
2048.0 2048.0  0.0    0.0   345088.0 150427.8 1397248.0   119834.5  80640.0 78303.8 84
48.0 7919.0    133    8.230   5     11.160   19.390
```

```shell
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）  
EC、EU：Eden区容量和使用量  
OC、OU：年老代容量和使用量  
PC、PU：永久代容量和使用量  
YGC、YGT：年轻代GC次数和GC耗时  
FGC、FGCT：Full GC次数和Full GC耗时  
GCT：GC总耗时
```





## Java IO

###### Java中的NIO，BIO，AIO分别是什么#

- BIO:同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
- NIO:同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
- AIO:异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理.AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。


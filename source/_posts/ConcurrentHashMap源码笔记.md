---
title: ConcurrentHashMap源码笔记
date: 2019-06-27 11:09:41
tags:
  - ConcurrentHashMap
categories:
  - 源码解析
---
## spread
假设table的长度为n=2^k，取模操作hash%n等价于hash&(n-1)，n-1为mask(二进制的k-1个1)
即，hash的低k位决定了桶的位置，k位以上的高位不起作用，如果不同hash的低k位相同，就会产生碰撞

```java
    static final int spread(int h) {
        // 将高16位与低16位异或，增加hash的分散度，降低碰撞概率
        // 与上HASH_BITS，将最高位置为0，使spread结果为正数
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```

## put
循环做4件事情：
1. 如果表为空，初始化表，继续
2. 如果对应下标节点为空，cas插入，跳出
3. 如果发现表正在迁移，帮助迁移，继续
4. 执行put操作。如果链表长度超过阈值，转为树。如果为更新，直接返回，否则跳出

循环退出后，计数+1
```java
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /**
     * Implementation for put and putIfAbsent
     * onlyIfAbsent: true 只替换为null值的node, false: 都替换
     */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        // key value 不能为null
        if (key == null || value == null) throw new NullPointerException();
        // 计算hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        // 循环执行
        for (Node<K, V>[] tab = table; ; ) {
            Node<K, V> f;
            int n, i, fh;
            // 如果table为空，初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
                // 如果对应下标节点为空，直接插入
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // cas替换
                if (casTabAt(tab, i, null,
                        new Node<K, V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
                // 如果节点不为空，且节点的hash为-1，-1表示在迁移，则帮助迁移
            } else if ((fh = f.hash) == MOVED)
                // 此处，helpTransfer返回的tab为f的nextTable，或者为已完整迁移的新table
                tab = helpTransfer(tab, f);
                // 执行put操作
            else {
                V oldVal = null;
                // 请求同步锁，避免并发写操作
                synchronized (f) {
                    // double check
                    if (tabAt(tab, i) == f) {
                        // 链表结构
                        if (fh >= 0) {
                            // 统计节点个数（非精确）-- 统计的是原来链表的长度
                            // for中break之后，不会再执行++binCount
                            // f的binCount为1，第二个节点binCount为2
                          // 假设第k节点key相同，替换新值后break，binCount为k，链表长度未知
                          // 或者第k节点的next为空，链接上新节点后break，binCount为k，链表长度为k+1
                            binCount = 1;
                            for (Node<K, V> e = f; ; ++binCount) {
                                K ek;
                                // 找到相同的key，更新value
                                if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                                (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 否则添加到链表尾部
                                Node<K, V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K, V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        } else if (f instanceof TreeBin) {
                            // 红黑树结构，TreeBin的hash为-2
                            Node<K, V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    // 如果链表节点个数大于阈值，将链表转化为红黑树
                    // 如果原结构为红黑树，binCount=2
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    // oldVal不为null，说明是更新操作，节点个数无变化，直接返回
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 到这里说明是添加操作，计数+1
        addCount(1L, binCount);
        return null;
    }
```

## initTable
只有一个线程能进行初始化，其他线程让出CPU
```java
    /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    private final Node<K, V>[] initTable() {
        Node<K, V>[] tab;
        int sc;
        while ((tab = table) == null || tab.length == 0) {
            // sc小于0，说明正在扩容或者迁移，让出cpu
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                // 当前线程获得初始化机会，cas sizeCtl为-1
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        // 如果sizeCtl有正值，用正值，否则采用默认容量
                        // sizeCtl什么时候有正值？有什么样的正值？
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                        // 此时tab=table!=null，其他线程从本方法的循环中跳出
                        table = tab = nt;
                        // 相当于n - n*1/4，即sc = 0.75*n
                        // >>> 无符号右移
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 初始化完成后，sizeCtl=0.75*n
                    sizeCtl = sc;
                }
                // 初始化完成跳出循环
                break;
            }
        }
        return tab;
    }
```

## addCount

对于check，从putVal过来的几种情形（这里check=binCount）

1. table中槽为空，直接放入新节点，**check=0**
2. table中槽里的节点（称为first节点）key和新值一样，被替换，check=1 -- 这种情况在putVal直接return了，不会进到addCount
3. 非first节点的key和新值一样，或者加入了新节点，**check>1** -- 替换的也return了，只有新节点才进入addCount，这是check=binCount=原链表长度
4. 槽中是红黑树，**check=2**

总结下，这里check的值有0、k（>1）、2

```java
    /**
     * Adds to count, and if table is too small and not already
     * resizing, initiates transfer. If already resizing, helps
     * perform transfer if work is available.  Rechecks occupancy
     * after a transfer to see if another resize is already needed
     * because resizings are lagging additions.
     *
     * @param x the count to add
     * @param check if <0, don't check resize, if <= 1 only check if uncontended
     */    
    private final void addCount(long x, int check) {
        CounterCell[] as;
        long b, s;
        // 如果计数表不为空
        // 或者cas baseCount失败，说明存在并发竞争
        if ((as = counterCells) != null ||
                !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a;
            long v;
            int m;
            // 假设无竞争
            boolean uncontended = true;
            // 如果计数表为空
            // 如果计数表长度小于1
            // 如果计数表随机位为null
            // 如果随机位不为空，且cas替换计数失败，说明有竞争
            if (as == null || (m = as.length - 1) < 0 ||
                    (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                    !(uncontended =
                            U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                // 大概作用是加入了一个新的计数cell
              // 上述if第4个cas失败了，在fullAddCount会重新生成一个随机数，再把统计放入对应计数位
                fullAddCount(x, uncontended);
                return;
            }
            // if <0, don't check resize, if <= 1 only check if uncontended
            if (check <= 1)
                return;
            // 累加baseCount和计数表里的值
            s = sumCount();
        }
        // 判断是否要扩容
        if (check >= 0) {
            Node<K, V>[] tab, nt;
            int n, sc;
            // 如果节点个数大于阈值，0.75n，类似于加载因子
            // 并且table不为空 && table长度未超过上限
            while (s >= (long) (sc = sizeCtl) && (tab = table) != null &&
                    (n = tab.length) < MAXIMUM_CAPACITY) {
                // 获取表长度的一个标识值
                int rs = resizeStamp(n);
                // 如果在扩容，判断是否需要帮助迁移
                if (sc < 0) {
                    // 如果sc的高16位与标识符不等
                    // bug report: https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8214427
                    // sc == rs + 1，存在bug，正确判断为 sc == (rs << RESIZE_STAMP_SHIFT + 1)，判断已无扩容线程
                    // sc == rs + MAX_RESIZERS，正确判断为 sc == (rs << RESIZE_STAMP_SHIFT + MAX_RESIZERS)，判断扩容线程数已达最大
                    // 如果临时表nextTable为空，或者迁移下标transferIndex小于0，说明扩容结束
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                            transferIndex <= 0)
                        // 无需帮助扩容
                        break;
                    // 否则，将sc cas为sc+1，表示sc低16位加1，即扩容线程数增加了一个
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        // 扩容
                        transfer(tab, nt);
                }
                // 不处于扩容状态，cas sizeCtl为(rs << RESIZE_STAMP_SHIFT) + 2)，标识有1个线程在扩容
                // sizeCtl的高16位存储待扩容表长n的标识符，低16位存储[扩容线程数+1]
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                        (rs << RESIZE_STAMP_SHIFT) + 2))
                    // 扩容
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```
sizeCtl的注释说明，当sizeCtl为负数时，-1标识表初始化，-(sizeCtl-1)标识活动的扩容线程数
为什么在具体实现里，是sizeCtl的低16位，且需减去1的值，标识扩容线程数呢？
((rs << RESIZE_STAMP_SHIFT) + 2)这里有个疑问，为什么是+2？不能是+1么？

-- 无责任猜测是版本变更了，最初也许没有RESIZE_STAMP_SHIFT、没有高低16位这么复杂

如果是+1，假如表长为0，第一个扩容线程加入，sizeCtl=`[1.....0](16位)+[0....1](16位)`=很负的一个负数，除非sizeCtl溢出了，不然也没发现其他情况下有重复的情况

+2的话，sizeCtl=`[1.....0](16位)+[0....10](16位)`=还是很负的一个负数，和+1只有最低位有区别。

```
    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
    private transient volatile int sizeCtl;
```


## resizeStamp
resizeStamp的结果作为扩容&迁移时sizeCtl的高16位信息
1. sizeCtl为负数
2. 标识着此次扩容&迁移对应的表长n
```java
    static final int resizeStamp(int n) {
        // n的二进制前导0个数。因为表长n为2的次幂，每次扩容*2，意味着每次扩容前导0个数少1，用于判断是否为同一次扩容
        // 将第16位或为1，是为了左移RESIZE_STAMP_SHIFT后为负数
        // 1 << (RESIZE_STAMP_BITS - 1) = (0b)1000 0000 0000 0000
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
```

## helpTransfer
```java
    /**
     * Helps transfer if a resize is in progress.
     */
    final Node<K, V>[] helpTransfer(Node<K, V>[] tab, Node<K, V> f) {
        Node<K, V>[] nextTab;
        int sc;
        // 重新判断，旧表不为空，f为迁移节点，f关联的新表不为空
        if (tab != null && (f instanceof ForwardingNode) &&
                (nextTab = ((ForwardingNode<K, V>) f).nextTable) != null) {
            // 同addCount，循环判断是否需要帮助扩容
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                    (sc = sizeCtl) < 0) {
                // 不满足帮助扩容条件，跳出
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                // 否则，将sc低位cas+1（标识多一个线程扩容）成功后，帮助扩容
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            // 返回新表
            return nextTab;
        }
        // 返回，此时table已是完成扩容的表
        return table;
    }
```

## transfer
```java
    private final void transfer(Node<K, V>[] tab, Node<K, V>[] nextTab) {
        int n = tab.length, stride;
        // 计算步长，最短为16，最长为n（单CPU）
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        // 如果新表为空，先建新表，容量为2n
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            // 从后向前迁移
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K, V> fwd = new ForwardingNode<K, V>(nextTab);
        // 迁移推进标识，为false时跳出循环
        // 标识是否进行迁移范围分配
        boolean advance = true;
        // 全表迁移状态标识，true标识全部迁移完成
        boolean finishing = false; // to ensure sweep before committing nextTab
        // 下标i,bound赋予初值，循环中会计算本次迁移的范围
        for (int i = 0, bound = 0; ; ) {
            Node<K, V> f;
            int fh;
            // 分配循环
            while (advance) {
                int nextIndex, nextBound;
                // i未到达本次迁移下界bound，或者全表迁移完成，标识停止推进，不会走到else if的范围分配
                if (--i >= bound || finishing)
                    advance = false;
                 // transferIndex小于0，没有可分配的迁移了，标识停止推进
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                 // 否则，若cas transferIndex成功（减去步长），分配迁移范围
                } else if (U.compareAndSwapInt
                        (this, TRANSFERINDEX, nextIndex,
                                nextBound = (nextIndex > stride ?
                                        nextIndex - stride : 0))) {
                    // 下界bound为更新后的transferIndex
                    bound = nextBound;
                    // 上界i为之前的transferIndex减1
                    i = nextIndex - 1;
                    // 标识停止分配范围
                    advance = false;
                }
            }
            // i的临界判断
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 如果全表已迁移完成，赋值table和sizeCtl
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                // i已到临界条件，本线程迁移工作完成，cas将sizeCtl-1
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // 如果本线程是迁移工作中的最后一个活动线程，直接返回（sc为sizeCtl cas前的值）
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    // 如果还有其他线程在迁移，仅标识迁移完成，且推进继续（为了double check？check什么？）
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 以下为迁移处理
            // 如果节点为null，直接cas替换为fwd，成功则advance为true，重新分配
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 如果节点本身是fwd，说明本段步长已处理过，在while中重新分配范围
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            // 桶的头节点加锁，迁移到新表
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K, V> ln, hn;
                        // 链表结构
                        if (fh >= 0) {
                            int runBit = fh & n;
                            // 假设n=2^k，按照hash第k位为0或1分为2组，0组放低位，1组放高位
                            // lastRun之后的节点直接用原节点，不新new
                            // lastRun之前的节点在新表中逆序，之后的节点保持原序
                            Node<K, V> lastRun = f;
                            for (Node<K, V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            // 这里主要是判断lastRun的一串节点，要放高位还是低位
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            } else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 重新遍历链表，链接出0组和1组节点，lastRun已链接上某个组，无需再遍历
                            for (Node<K, V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash;
                                K pk = p.key;
                                V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K, V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K, V>(ph, pk, pv, hn);
                            }
                            // 替换
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            // 继续推进
                            advance = true;
                        }
                        // 树结构
                        else if (f instanceof TreeBin) {
                            TreeBin<K, V> t = (TreeBin<K, V>) f;
                            TreeNode<K, V> lo = null, loTail = null;
                            TreeNode<K, V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K, V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K, V> p = new TreeNode<K, V>
                                        (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                } else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                    (hc != 0) ? new TreeBin<K, V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                    (lc != 0) ? new TreeBin<K, V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

## get

```java
    public V get(Object key) {
        Node<K, V>[] tab;
        Node<K, V> e, p;
        int n, eh;
        K ek;
        int h = spread(key.hashCode());
        // 表不为空，且hash取模所在桶不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (e = tabAt(tab, (n - 1) & h)) != null) {
            // 桶的头结点为要找的节点
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            } 
            // 头结点hash<0，说明为树或者迁移节点，调用find查找
            else if (eh < 0)
                // ForwardingNode -1; treeBin -2
                return (p = e.find(h, key)) != null ? p.val : null;
            // 否则为链表格式，遍历查找
            while ((e = e.next) != null) {
                if (e.hash == h &&
                        ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```



## 作者说
```
     ....We do not want to waste
     * the space required to associate a distinct lock object with
     * each bin, so instead use the first node of a bin list itself as
     * a lock. Locking support for these locks relies on builtin
     * "synchronized" monitors.
     * 
     * Using the first node of a list as a lock does not by itself
     * suffice though: When a node is locked, any update must first
     * validate that it is still the first node after locking it, and
     * retry if not. ....
     * 
     * ....The transfer operation must also ensure that all
     * accessible bins in both the old and new table are usable by any
     * traversal.  This is arranged in part by proceeding from the
     * last bin (table.length - 1) up towards the first.  
```

## 引用一段并发分析

[Java 8 中 ConcurrentHashMap工作原理的要点分析](https://www.bbsmax.com/A/ZOJPOX7xzv/)

> 6.1初化的同步问题

> 表长度的分配并不是在构造函数中进行的，而是在put方法中进行的，也就是说这实际上是个懒汉模式。但是如果多个线程同时进行表长度的空间分配，显然是非线程安全的。所以只能有一个线程来进行创建表，其它线程会等待创建完成。ConcurrentHashMap类中设定一个volatile变量sizeCtl

    private transient volatile int sizeCtl;
> 然后通过CAS方法去修改它，如果有其它线程发现sieCtl为-1


    U.compareAndSwapInt(this, SIZECTL, sc, -1)
> 就表示已经有线程正在创建表了，那么当前线程就会放弃CPU使用权（调用Thread.yield()方法），等待分初始化完成后继续进行put操作。否则当前线程尝试将siezeCtl修改为-1,若成功，就由当前线程来创建表。

> 6.2 put方法和remove方法之间的同步问题

> 在表的同一个槽上，一个线程调用put方法和另一个线程调用put方法是互斥的；在表的同一个槽上，一个线程调用remove方法和另一个线程调用remove方法也是互斥的；在表的同一个槽上，一个线程调用remove方法和另一个线程调用put方法也是互斥的。这些互斥操作在代码中都是通过锁来保证的。

> 6.3 put(或remove)方法和get方法的同步问题


> 实际上是不需要同步，先到先得。这主要由于Node定义中value和next都定义成了volatile类型。一个线程能否get到另一个线程刚刚put（或remove）的值，这主要由两个线程当前访问的结点所处的位置决定的。

> 6.4 get方法和扩容操作的同步问题

> 可以分成两种情况讨论

> 1）该位置的头结点是Node类型对象，直接get，即使这个桶正在进行迁移，在get方法未完成前，迁移已完成（槽被设置成了ForwordingNode对象），也没关系，并不影响get的结果，因为get线程仍然持有旧链表的引用，可以从当前结点位置访问到所有的后续结点，原因是新表中的节点是通过复制旧表中的结点得到的，所以新表的结点的next不会影响旧表中对应结点的next值。当get方法结束后，旧链表就不可达了，会被垃圾回收线程回收。

> 2）该位置的头结点是ForwordingNode类型对象（头结点的hash值 == -1），头结点是ForwordingNode类型的对象，调用该对象的find方法，在新表中查找。

> 所以无论哪种情况，都能get到正确的值。

> 6.5 put(或remove)方法和扩容操作的同步问题

> 同样可以分为两种情况讨论：

> 1）该位置的头结点是Node类型对象，put操作就走正常路线，先将Node对象放入到旧表中，然后调用addCount方法，判断是否需要帮助扩容。

> 2）该位置的头结点是ForwordingNode类型对象，那就会先帮助扩容，然后在新表中进行put操作。
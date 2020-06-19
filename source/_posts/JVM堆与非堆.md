---
title: JVM堆与非堆
tags:
  - null
categories:
  - null
date: 2020-05-07 18:33:01
---



# 堆和非堆内存
|             |    |
| :---------- | -------------- |
| Heap memory | Code Cache     |
|             | Eden Space 新生代 |
|             | Survivor Space 新生代 |
|             | Tenured Gen 老年代 |
| non-heap memory | Perm Gen 永久代 |
|  | native heap?(I guess) |







# 内存监控方法

- jamp -heap pid 查看堆内存使用情况

```shell
jmap -heap 212
Attaching to process ID 212, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.20-b23

using thread-local object allocation.
Parallel GC with 43 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4294967296 (4096.0MB)
   NewSize                  = 1431306240 (1365.0MB)
   MaxNewSize               = 1431306240 (1365.0MB)
   OldSize                  = 2863661056 (2731.0MB)
   NewRatio                 = 2 # 老年代与新生代大小比值
   SurvivorRatio            = 8 # Eden与Survivor大小比值
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1414529024 (1349.0MB)
   used     = 496589416 (473.5845718383789MB)
   free     = 917939608 (875.4154281616211MB)
   35.10634335347508% used
From Space:
   capacity = 8388608 (8.0MB)
   used     = 5778216 (5.510536193847656MB)
   free     = 2610392 (2.4894638061523438MB)
   68.8817024230957% used
To Space:
   capacity = 8388608 (8.0MB)
   used     = 0 (0.0MB)
   free     = 8388608 (8.0MB)
   0.0% used
PS Old Generation
   capacity = 2863661056 (2731.0MB)
   used     = 125500192 (119.68630981445312MB)
   free     = 2738160864 (2611.313690185547MB)
   4.382508598112527% used

40327 interned Strings occupying 4296152 bytes.
```



# JVM收集器设置

```shell
–XX:+UseSerialGC
–XX:+UseParallelGC
–XX:+UseParallelOldGC
–XX:+UseConcMarkSweepGC
```

1. Serial Collector

   大部分平台或者加java -client参数

   young generation算法 = serial

   old generation算法 = serial (mark-sweep-compact)

   缺点：串行，stop-the-world，速度慢，服务器不推荐使用

2. Parallel Collector

   linux x64平台，或加java -server参数

   young = parallel，多个thread同时copy

   old = mark-sweep-compact = 1

   优点：新生代回收更快，提高了吞吐量throughput（cpu用于非gc时间）

   缺点：运行在8G/16G server上，old generation live obeject太多，pause time 过长（堆内存大->老年代大->能放的存活对象多）

3. Parallel Compact Collector（ParallelOld）

   young = parallel = 2

   old = parallel，分成多个独立单元，如果单元中存活对象少就回收，多则跳过

   优点：old generation性能较2有提高

   缺点：【大部分server系统old generation内存占用会达到60%-80%, 没有那么多理想的单元live object很少方便迅速回收，同时compact方面开销比起parallel并没明显减少。】

4. Concurrent Mark-Sweep(CMS) Collector

   young = parallel = 2

   old = cms

   【同时不做compact操作】

   优点：pause time 会降低，pause敏感但cpu有空闲的场景建议使用4

   缺点：cpu占用过多，cpu密集型服务不适合使用。另外碎片多，object的存储要通过链表连续跳n个地方，空间浪费问题也会增大。





# 参考

1. [JVM堆内存和非堆内存](http://xstarcd.github.io/wiki/Java/JVM_Heap_Non-heap.html)
2. [JVM参数设置](https://www.cnblogs.com/ceshi2016/p/8447989.html)
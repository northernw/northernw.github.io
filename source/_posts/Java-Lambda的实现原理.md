---
title: Java-Lambda的实现原理
tags:
  - INBOX
  - Java
  - Lambda
categories:
  - INBOX
date: 2020-11-09 18:27:57
---



**函数接口（Functional Interface）**

并不能在代码的任何地方任意写Lambda表达式。

Lambda的类型，是对应函数接口的类型。

省略了接口名、方法名、参数类型（基于类型推断）。

> 将功能视为方法参数。



###### 匿名内部类与Lambda实现对比

匿名内部类会产生额外的class文件

Lambda被封装成主类的一个私有方法，并通过invokedynamic指令进行调用。



###### 自定义函数接口

编写一个只有一个抽象方法的接口。

`@FunctionalInterface`是可选的，加上注解编译器会帮忙检查是否符合函数接口的规范。（类似`@Override`会帮助检查是否重载）





##### 函数式数据处理

###### 使用流

1. 筛选和切片

   1. filter 谓词筛选
   2. distinct 筛选各异的元素，去重
   3. limit 截断流，返回前n个元素
   4. skip 跳过元素，跳过前n个元素

2. 映射

   1. map
   2. flatMap 扁平化流

3. 查找和匹配

   1. anyMatch 至少一个匹配
   2. allMatch 完全匹配
   3. noneMatch 都不匹配
   4. findAny 返回任意元素
   5. findFirst 返回第一个元素

4. 规约

   1. reduce (折叠，fold)
   2. 最大值 `.reduce(Integer::max)`
   3. 最小值 `.reduce(Integer::min)`

5. 原始类型流特化

   1. IntStream、DoubleStream、LongStream，分别将流中的元素转化为int、long、double，避免暗含的装箱成本

      1. mapToInt.. 将Stream转换为数值流

      2. boxed() 装箱，将数值流转为Stream

      3. ```java
         IntStream intstream = menu.stream().mapToInt(Dish::getCalories);// 将Stream转换为数值流
         Stream<Integer> stream = intStream.boxed(); // 将数值流转为Stream
         ```

   2. 常用数值规约

      1. sum
      2. max
      3. min
      4. OptionalInt

   3. 数值范围

      1. rangeClosed 闭区间，起始值，结束值

      2. range 开区间，起始值，结束值（开区间）

6. 从多个源创建流

   1. 集合、值、数组、文件，以及5里的迭代和生成

7. 无限流，没有固定的大小

   1. 迭代 

      ```java
      Stream.iterate(0,n->n+2).limit(10).forEach(System.out::println);
      ```

   2. 生成

      ```java
      Stream.generate(Math::random).limit(5).forEach(System.out::println);
      ```




###### 收集数据

1. counting 统计个数
2. maxBy minBy 最大值、最小值
3. 汇总 summingInt/summingLong/summingDouble
4. 平均值 averagingInt/...
5. 连接字符串 joining
6. 广义规约 reducing，有三个参数
   1. 起始值
   2. 提供数值的函数
   3. BinaryOperator 进行规约的二元操作
7. groupingBy 分组
8. 按子组收集数据 `Map<Dish.Type, Long> typesCount = ...(groupingBy(Dish::getTyype, counting()))`



###### Optional

1. `isPresent()` Optional存在值时返回true，否则false
2. `ifPresent(Consumer<T> block)` 会在值存在时执行给定代码块
3. `T get()` 会在值存在时返回值，否则抛出一个`NoSuchElement`异常
4. `T orElse(T other)` 会在值存在时返回值，否则返回一个默认值
---
title: Dubbo学习笔记
tags:
  - null
categories:
  - null
date: 2019-10-16 15:53:29
---





架构图

<img src="/github/northernw.github.io/image/image-20191016155539701.png" alt="image-20191016155539701" style="zoom:50%;" />



dubbo关键功能
1. 基于rpc的远程调用
2. 容错与负载均衡
3. 服务注册与发现

[一些面试题-dubbo面试题、基本原理、核心配置]: http://baijiahao.baidu.com/s?id=1601781224676099887&amp;wfr=spider&amp;for=pc
[官网]: http://dubbo.apache.org



# 学习笔记

《深度剖析ApacheDubbo核心技术内幕》翟陆续.2019.电子工业出版社





























---------

《深入理解Apache Dubbo与实战》诣极 林琳@著



## 第4章 Dubbo扩展点加载机制

### 注解

一、@SPI

> 在某个接口上加上@SPI注解后，表明该接口为可扩展接口。

`@SPI`的value是一个key，指定这个接口的默认实现是什么。

用户在配置文件中，可指定自己想要的实现。如`Protocal`可通过`<dubbo:protocol />`指定（这样就不一定是用@SPI里的`key`对应的实现了）。

二、@Adaptive

> 保证dubbo在内部调用具体实现的时候不是硬编码来指定引用哪个实现，适配一个接口的多种实现

如果注解在方法上，动态获取键值对指定的实现，再执行实现的相应方法。编译时，为这个接口生成一个实现类（策略适配器），带有`@Adaptive`注解的方法里会增加一段类似策略选择的代码，比如`Transporter`接口的`bind`方法，将注解的`server`作为key去URL中找相应value，value值对应的类是本次调用需要的真正实现。

如果注解在类上，就以这个类作为接口的策略适配器。

**可以理解为，`@Adaptive`标识或产生一个策略适配器。用户在调用接口时，传入key指定需要的实现是哪一个。**

三、@Activate

自动激活带有这个注解的所有实现。



@SPI在接口上，@Adaptive在接口的方法或者一个类上，@Activate在实现类上。



### ExtensionLoader

每一个SPI接口，都有一个ExtensionLoader?

用来获取普通扩展类、自适应扩展类、自动激活的扩展类。

一般调用方是谁？----要用到某个接口的实现的地方，可以获取指定的一个实现、自适应的实现（适配器）、或者多个实现（比如filter等）。



### ExtensionLoaderFactory

目测没有太大作用。在ExtensionLoader的`injectExtension`中有调用，依赖注入时利用factory去getExtension。

其实注入是用ExtensionLoader的getExtension也能实现相同功能，用factory是可以注入spring容器托管的bean。

dubbo也有自己的一个spi容器。

ExtensionLoaderFactory有三个实现，分别是adaptive、spi、spring.









参考：

[Dubbo源码解析（二）Dubbo扩展机制SPI]: https://segmentfault.com/a/1190000016842868
[dubbo源码分析4——SPI机制_ExtensionFactory类的作用]: https://www.cnblogs.com/hzhuxin/p/7553070.html








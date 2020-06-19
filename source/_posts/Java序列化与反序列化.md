---
title: Java序列化与反序列化
tags:
  - null
categories:
  - null
date: 2020-06-19 16:45:34
---

序列化与反序列化是成对存在的，文中简称为序列化



# 简介

**序列化（serialize）** - 序列化是将对象转换为字节流。

**反序列化（deserialize）** - 反序列化是将字节流转换为对象。

**序列化用途**

- 序列化可以将对象的字节序列持久化——保存在内存、文件、数据库中
- 在网络上传送对象
- RMI（远程方法调用）



# 序列化工具

java序列化 - 问题比较多

[thrift](https://github.com/apache/thrift)、[protobuf](https://github.com/protocolbuffers/protobuf) - 适用于对性能敏感，对开发体验要求不高的内部系统。

[hessian](http://hessian.caucho.com/doc/hessian-overview.xtp) - 适用于对开发体验敏感，性能有要求的内外部系统。

[jackson](https://github.com/FasterXML/jackson)、[gson](https://github.com/google/gson)、[fastjson](https://github.com/alibaba/fastjson) - 适用于对序列化后的数据要求有良好的可读性（转为 json 、xml 形式）。



## Java序列化

[Java序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#heading-8)



### 使用

JavaBean实现Serializable接口或者Externalizable接口。

普通序列化

1. 实现Serializable接口
2. 序列化：ObjectOutputStream#writeObject
3. 反序列化：ObjectInputStream#readObject
4. 如果属性是引用对象，引用对象也要实现序列化
5. transient关键字修饰的对象不参与序列化
6. 在类中重写writeObject和readObject可以自定义序列化逻辑，比如控制序列化字段、默认值、进行编码加密等
7. 同一对象在一个ObjectOutputStream里writeObject多次，后面几次只会输出序列化编码号
8. **潜在问题：**如果在第一次序列化后，修改了内容，再次序列化时只会输出编码号，会丢失修改信息。

Externalizable序列化

强制重写方法，必须实现writeExternal、readExternal方法。

必须提供pulic的无参构造器，因为在反序列化的时候需要反射创建对象。



https://juejin.im/post/5c94584fe51d4508ae01ccf6

**`serialVersionUID` 是 Java 为每个序列化类产生的版本标识**。它可以用来保证在反序列时，发送方发送的和接受方接收的是可兼容的对象。如果接收方接收的类的 `serialVersionUID` 与发送方发送的 `serialVersionUID` 不一致，会抛出 `InvalidClassException`。

如果可序列化类没有显式声明 `serialVersionUID`，则序列化运行时将基于该类的各个方面计算该类的默认 `serialVersionUID` 值。尽管这样，还是**建议在每一个序列化的类中显式指定 `serialVersionUID` 的值**。因为不同的 jdk 编译很可能会生成不同的 `serialVersionUID` 默认值，从而导致在反序列化时抛出 `InvalidClassExceptions` 异常。

**`serialVersionUID` 字段必须是 `static final long` 类型**。



[一些实现分析](https://juejin.im/post/5b4c69dcf265da0fa959aa06#heading-3)



为什么要实现Serializable接口？

Serializable是一个标记接口，没有方法。



### 原理

[Java序列化系列](https://www.cnblogs.com/binarylei/category/1159503.html)

反射获取对象的Field。

本质上是先写对象信息，再写对象每个属性的信息。是一个递归的过程，递归到最后是写基本类型的数据。

String的调用栈

![image-20200619200736381](/github/northernw.github.io/image/image-20200619200736381.png)

Integer的调用栈

![image-20200619201836237](/github/northernw.github.io/image/image-20200619201836237.png)



## Json序列化

### Gson

###Jackson

### FastJson 



## 其他序列化



# 性能对比


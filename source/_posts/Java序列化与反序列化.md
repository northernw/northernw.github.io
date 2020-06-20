---
title: Java序列化与反序列化
tags:
  - null
categories:
  - null
date: 2020-06-19 16:45:34
---

**序列化与反序列化是成对存在的，文中简称为序列化。**



# 简介

**序列化（serialize）** - 序列化是将对象转换为字节流。

**反序列化（deserialize）** - 反序列化是将字节流转换为对象。

**序列化用途**

- 将对象持久化，保存在内存、文件、数据库中
- 便于网络传输和传播



# 序列化工具

java序列化 - 性能比较普通

[thrift](https://github.com/apache/thrift)、[protobuf](https://github.com/protocolbuffers/protobuf) - 适用于对性能敏感，对开发体验要求不高的内部系统。

[hessian](http://hessian.caucho.com/doc/hessian-overview.xtp) - 适用于对开发体验敏感，性能有要求的内外部系统。

[jackson](https://github.com/FasterXML/jackson)、[gson](https://github.com/google/gson)、[fastjson](https://github.com/alibaba/fastjson) - 适用于对序列化后的数据要求有良好的可读性（转为 json 、xml 形式）。



## Java序列化

实现Serializable或Externalizable接口。

前者有默认序列化逻辑，后者需要强制实现自己的序列化逻辑。底层实现都是Serializable。

本文主要关注Serializable。

### 使用

#### 基础用法

实现Serializable接口，通过ObjectOutputStream来序列化，ObjectInputStream反序列化。

```java
@Slf4j
@Data
public class Person implements Serializable {
    private static final long serialVersionUID = 6666716291353949192L;
    private Integer age;
    private String name;

    @Test
    public void test() throws IOException, ClassNotFoundException {
        Person person = new Person();
        person.setAge(22);
        person.setName("lily");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("java.person.txt"));
        objectOutputStream.writeObject(person);
        log.info("writeObject = {}", person);

        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("java.person.txt"));
        person = (Person) objectInputStream.readObject();
        log.info("readObject = {}", person);
    }
}
```

输出

```shell
writeObject = Person(age=22, name=lily)
readObject = Person(age=22, name=lily)
```



#### 自定义用法

自定义序列化和反序列化逻辑，比如控制序列化字段、进行编码加密等。

在JavaBean中实现writeObject和readObject方法，方法签名是固定的。序列化过程中会判断JavaBean中的有无这样的方法，如果没有，就走入默认的defaultWriteFields和defaultReadFields的逻辑。

比如用age模拟加密和解密。

```java
    private void writeObject(ObjectOutputStream out) throws Exception {
        ObjectOutputStream.PutField putFields = out.putFields();
        putFields.put("age", age + 1);
        out.writeFields();
    }

    private void readObject(ObjectInputStream in) throws Exception {
        ObjectInputStream.GetField readFields = in.readFields();
        Integer encryptionAge = (Integer) readFields.get("age", "");
        // 模拟解密
        age = encryptionAge - 1;
    }
```



#### 其他知识点

1. 如果不实现Serializable接口，会抛出NotSerializableException异常。

2. 如果属性是引用对象，引用对象也要实现序列化

3. 一个子类实现Serializable 接口，父类也需要实现（否则父类信息不会被序列化）

4. 静态字段不会参与序列化（序列化保存的是对象的状态，静态变量属于类的状态）

5. transient关键字修饰的对象不参与序列化

6. 第二种自定义方式：writeReplace和readResolve

   1. writeObject 序列化时会调用 writeReplace 方法将当前对象替换成另一个对象并将其写入流中，此时如果有自定义的 writeObject 也不会生效了
   2. readResolve 会在 readObject 调用之后自动调用，它最主要的目的就是对反序列化的对象进行修改后返回

7. 同一对象在一个ObjectOutputStream里writeObject多次，后面几次只会输出句柄信息

8. **潜在问题：**如果在第一次序列化后，修改了内容，再次序列化时只会输出编码号，会丢失修改信息

9. serialVersionUID相当于类的版本号，如果没有显示定义serialVersionUID，编译期会根据类信息创建一个，当修改类后可能会引起反序列化失败（比如新增了字段，导致再次自动计算的serialVersionUID不相同）

10. serialVersionUID的idea快捷生成

    ![image-20200620173518028](/github/northernw.github.io/image/image-20200620173518028.png)



### 实现原理

#### 接口关系

![Java序列化接口](/github/northernw.github.io/image/1322310-20190606081015449-98486965.png)

1. `Serializable`和`Externalizable` 序列化接口

   Serializable 接口没有方法或字段，仅用于标识可序列化的语义，实际上 ObjectOutputStream#writeObject 会判断JavaBean有没有自定义的writeObject 方法，如果没有则调用默认的序列化方法。

   Externalizable 接口该接口中定义了两个扩展的抽象方法：writeExternal 与 readExternal。

2. `DataOutput`和`ObjectOutput` 

   DataOutput 定义了对 8种Java 基本类型 byte、short、int、long、float、double、char、boolean，以及 String 的操作。

   ObjectOutput 在 DataOutput 的基础上定义了对 Object 类型的操作。

3. `ObjectOutputStream` 

   一般使用 ObjectOutputStream#writeObject 方法把一个对象进行持久化。（ObjectInputStream#readObject 则从持久化存储中把对象读取出来。）

4. `ObjectStreamClass` 和 `ObjectStreamField` 

   ObjectStreamClass 是类的序列化描述符，包含类描述信息，字段的描述信息和 serialVersionUID。可以使用 lookup 方法找到创建在虚拟机中加载的具体类的 ObjectStreamClass。

    ObjectStreamField 保存字段的序列化描述符，包括字段名、字段值等。



#### ObjectOutputStream源码分析





小知识点：

ObjectOutputStream写值的逻辑：获取到当前对象中的原生类型字段，primDataSize是对象中所有原生类型字段的总长度，`desc.getPrimFieldValues(obj, primVals);`是将对象中的所有原生字段的值写入primVals这个字节数组，获取对象的值时是用Unsafe类直接通过偏移量取值，`unsafe.getBoolean(obj, offset)`这样的方式。

同理，ObjectInputStream读值的逻辑，`unsafe.putBoolean(obj, key, Bits.getBoolean(buf, off));`

实现在`ObjectStreamClass#getPrimFieldValues`和`ObjectStreamClass#setPrimFieldValues`



**以下自己看**

为什么要实现Serializable接口？

Serializable是一个标记接口，告诉程序只要是实现了Serializable接口的类都是可以被序列化的。

![image-20200620170911977](/github/northernw.github.io/image/image-20200620170911977.png)



### 原理

[Java序列化系列](https://www.cnblogs.com/binarylei/category/1159503.html)

反射获取对象的Field。

本质上是先写对象信息，再写对象每个属性的信息。是一个递归的过程，递归到最后是写基本类型的数据。

String的调用栈

![image-20200619200736381](/github/northernw.github.io/image/image-20200619200736381.png)

Integer的调用栈

![image-20200619201836237](/github/northernw.github.io/image/image-20200619201836237.png)



## Gson

### 使用



### 实现原理



### 小结



###Jackson

### FastJson 



## 其他序列化



# 性能对比





# 参考

1. [Java序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#heading-8)

2. [Java 序列化和反序列化（一）Serializable 使用场景](https://www.cnblogs.com/binarylei/p/10987540.html)
3. 


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

引用[这里](https://www.cnblogs.com/binarylei/p/10987933.html)一张接口关系图

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

源码版本：jdk-12.0.1.jdk

##### ObjectOutputStream 数据结构

```java
    /** 输出流 */
    private final BlockDataOutputStream bout;
    /** 句柄映射，如果在句柄中找到当前对象，说明已经序列化过，只输出句柄信息 */
    private final HandleTable handles;
    /** 替换对象的映射 obj -> replacement obj map */
    private final ReplaceTable subs; 
    /** true 则调用writeObjectOverride()来替代writeObject() -- ObjectOutputStream的子类可重写writeObjectOverride()*/
    private final boolean enableOverride;
    /** true 则调用replaceObject() -- JavaBean中实现replaceObject() */
    private boolean enableReplace;
```



##### ObjectOutputStream 构造函数

```java
    /** 一些初试化 */
    public ObjectOutputStream(OutputStream out) throws IOException {
        verifySubclass();
        bout = new BlockDataOutputStream(out);
        handles = new HandleTable(10, (float) 3.00);
        subs = new ReplaceTable(10, (float) 3.00);
        enableOverride = false;
        writeStreamHeader();
        bout.setBlockDataMode(true);
        if (extendedDebugInfo) {
            debugInfoStack = new DebugTraceInfoStack();
        } else {
            debugInfoStack = null;
        }
    }
    /**
     *  魔数和版本号
     */
    protected void writeStreamHeader() throws IOException {
        bout.writeShort(STREAM_MAGIC);
        bout.writeShort(STREAM_VERSION);
    }
```



##### 序列化入口 writeObject

引用[这里](https://www.cnblogs.com/binarylei/p/10987933.html)一张时序图

![writeObject调用过程](/github/northernw.github.io/image/1322310-20190607214343020-892127930.png)

以下顺着基础用法的逻辑，看下代码实现。

###### 1. writeObject

```java
    public final void writeObject(Object obj) throws IOException {
      // 如果当前是ObjectOutputStream的子类，走这个分支
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
          // 核心方法
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }
```

###### 2. writeObject0

```java
    /**
     * Underlying writeObject/writeUnshared implementation.
     * unshared=false表示共享对象，就是指同一个对象只输出句柄信息
     * unshared=true表示不共享，会完整再序列化一次
     */
    private void writeObject0(Object obj, boolean unshared)
        throws IOException
    {
        boolean oldMode = bout.setBlockDataMode(false);
      // depth表示对象深度，比如当前对象为1，递归到对象属性时，depth++为2
        depth++;
        try {
            // handle previously written and non-replaceable objects
            int h;
          // 判断要不要序列化 以下4种情况不用序列化
            if ((obj = subs.lookup(obj)) == null) {
                writeNull();
                return;
            } else if (!unshared && (h = handles.lookup(obj)) != -1) { // 是否共享已序列化对象，一般为是
                writeHandle(h);
                return;
            } else if (obj instanceof Class) {
                writeClass((Class) obj, unshared);
                return;
            } else if (obj instanceof ObjectStreamClass) {
                writeClassDesc((ObjectStreamClass) obj, unshared);
                return;
            }

            // check for replacement object
          // 这里处理对象替换，也先不看
            Object orig = obj;
          // ps最后回来看: 怎么处理泛型呢？类信息写入的类型是java.lang.Object；写数据时，字段对应的类型是实际类型，比如java.lang.Integer这样的
            Class<?> cl = obj.getClass();
            ObjectStreamClass desc;
            for (;;) {
                // REMIND: skip this check for strings/arrays?
                Class<?> repCl;
                desc = ObjectStreamClass.lookup(cl, true);
                if (!desc.hasWriteReplaceMethod() ||
                    (obj = desc.invokeWriteReplace(obj)) == null ||
                    (repCl = obj.getClass()) == cl)
                {
                    break;
                }
                cl = repCl;
            }
          // 如果对象替换了，取新对象的描述符
            if (enableReplace) {
                Object rep = replaceObject(obj);
                if (rep != obj && rep != null) {
                    cl = rep.getClass();
                    desc = ObjectStreamClass.lookup(cl, true);
                }
                obj = rep;
            }

            // 如果对象替换了，再判断一遍要不要序列化
            if (obj != orig) {
                subs.assign(orig, obj);
                if (obj == null) {
                    writeNull();
                    return;
                } else if (!unshared && (h = handles.lookup(obj)) != -1) {
                    writeHandle(h);
                    return;
                } else if (obj instanceof Class) {
                    writeClass((Class) obj, unshared);
                    return;
                } else if (obj instanceof ObjectStreamClass) {
                    writeClassDesc((ObjectStreamClass) obj, unshared);
                    return;
                }
            }

            // 序列化的主体逻辑在这里
            // 字符串和枚举在方法里写值进输出流了
            // 数组里的元素，如果是原生类型，也直接写输出流，如果非原生类型，递归序列化
            if (obj instanceof String) { // 字符串
                writeString((String) obj, unshared);
            } else if (cl.isArray()) { // 数组
                writeArray(obj, desc, unshared);
            } else if (obj instanceof Enum) { // 枚举
                writeEnum((Enum<?>) obj, desc, unshared);
            } else if (obj instanceof Serializable) { // 实现了Serializable的JavaBean，也是我们要主要看的逻辑
                writeOrdinaryObject(obj, desc, unshared);
            } else { // 不是以上几种情况的抛出异常
                if (extendedDebugInfo) {
                    throw new NotSerializableException(
                        cl.getName() + "\n" + debugInfoStack.toString());
                } else {
                    throw new NotSerializableException(cl.getName());
                }
            }
        } finally {
            depth--;
            bout.setBlockDataMode(oldMode);
        }
    }
```

###### 2.1 writeString&writeArray&writeEnum

字符串、枚举的在这里就写值了

数组元素如果是原生类型的，也写值了，如果非原生，递归调用writeObject0

```java
    /**
     * Writes given string to stream, using standard or long UTF format
     * depending on string length.
     * UTF格式写string
     */
    private void writeString(String str, boolean unshared) throws IOException {
        handles.assign(unshared ? null : str);
        long utflen = bout.getUTFLength(str);
        if (utflen <= 0xFFFF) {
            bout.writeByte(TC_STRING);
            bout.writeUTF(str, utflen);
        } else {
            bout.writeByte(TC_LONGSTRING);
            bout.writeLongUTF(str, utflen);
        }
    }

    /**
     * 写数组
     * Writes given array object to stream.
     */
    private void writeArray(Object array,
                            ObjectStreamClass desc,
                            boolean unshared)
        throws IOException
    {
        bout.writeByte(TC_ARRAY);
        writeClassDesc(desc, false);
        handles.assign(unshared ? null : array);

        Class<?> ccl = desc.forClass().getComponentType();
      // 如果是原生类型的数组
        if (ccl.isPrimitive()) {
            if (ccl == Integer.TYPE) {
                int[] ia = (int[]) array;
                bout.writeInt(ia.length);
                bout.writeInts(ia, 0, ia.length);
            } 
            // ... 省略其他原生类型 
            else {
                throw new InternalError();
            }
        } else {
            Object[] objs = (Object[]) array;
            int len = objs.length;
          // 写长度信息
            bout.writeInt(len);
            if (extendedDebugInfo) {
                debugInfoStack.push(
                    "array (class \"" + array.getClass().getName() +
                    "\", size: " + len  + ")");
            }
            try {
                for (int i = 0; i < len; i++) {
                    if (extendedDebugInfo) {
                        debugInfoStack.push(
                            "element of array (index: " + i + ")");
                    }
                    try {
                      // 对每个对象序列化
                        writeObject0(objs[i], false);
                    } finally {
                        if (extendedDebugInfo) {
                            debugInfoStack.pop();
                        }
                    }
                }
            } finally {
                if (extendedDebugInfo) {
                    debugInfoStack.pop();
                }
            }
        }
    }

    /**
     * Writes given enum constant to stream.
     */
    private void writeEnum(Enum<?> en,
                           ObjectStreamClass desc,
                           boolean unshared)
        throws IOException
    {
        bout.writeByte(TC_ENUM);
        ObjectStreamClass sdesc = desc.getSuperDesc();
      // 写枚举类信息
        writeClassDesc((sdesc.forClass() == Enum.class) ? desc : sdesc, false);
        handles.assign(unshared ? null : en);
      // 写枚举name
        writeString(en.name(), false);
    }
```



###### 2.2 writeOrdinaryObject

普通JavaBean的序列化逻辑

1. 写类型
2. 写类信息
3. 写类数据

```java
    /**
     * Writes representation of a "ordinary" (i.e., not a String, Class,
     * ObjectStreamClass, array, or enum constant) serializable object to the
     * stream.
     */
    private void writeOrdinaryObject(Object obj,
                                     ObjectStreamClass desc,
                                     boolean unshared)
        throws IOException
    {
        if (extendedDebugInfo) {
            debugInfoStack.push(
                (depth == 1 ? "root " : "") + "object (class \"" +
                obj.getClass().getName() + "\", " + obj.toString() + ")");
        }
        try {
            desc.checkSerialize();
          // 写类型
            bout.writeByte(TC_OBJECT);
          // 写类信息
            writeClassDesc(desc, false);
            handles.assign(unshared ? null : obj);
          // 写数据（Field信息和数据）
            if (desc.isExternalizable() && !desc.isProxy()) { // Externalizable接口
                writeExternalData((Externalizable) obj);
            } else {
                writeSerialData(obj, desc); // Serializable接口
            }
        } finally {
            if (extendedDebugInfo) {
                debugInfoStack.pop();
            }
        }
    }
```



###### 2.2.1 writeClassDesc

写类信息。

非代理类型的类信息，一般有类标志位、类名称、序列化协议版本、SUID、方法标志位、字段个数、然后每个字段的：字段TypeCode、字段名称、（非原生类型的）字段类型

```java
    private void writeClassDesc(ObjectStreamClass desc, boolean unshared)
        throws IOException
    {
        int handle;
        if (desc == null) {
            writeNull();
        } else if (!unshared && (handle = handles.lookup(desc)) != -1) {
            writeHandle(handle);
        } else if (desc.isProxy()) {
            writeProxyDesc(desc, unshared);
        } else {
            writeNonProxyDesc(desc, unshared);
        }
    }
     /**
     * Writes class descriptor representing a standard (i.e., not a dynamic
     * proxy) class to stream.
     */
    private void writeNonProxyDesc(ObjectStreamClass desc, boolean unshared)
        throws IOException
    {
        bout.writeByte(TC_CLASSDESC);
        handles.assign(unshared ? null : desc);

        if (protocol == PROTOCOL_VERSION_1) {
            // do not invoke class descriptor write hook with old protocol
            desc.writeNonProxy(this);
        } else {
            writeClassDescriptor(desc);
        }

        Class<?> cl = desc.forClass();
        bout.setBlockDataMode(true);
        if (cl != null && isCustomSubclass()) {
            ReflectUtil.checkPackageAccess(cl);
        }
        annotateClass(cl);
        bout.setBlockDataMode(false);
        bout.writeByte(TC_ENDBLOCKDATA);

      // 往上递归，获取父类信息，直到父类没有实现Serializable
        writeClassDesc(desc.getSuperDesc(), false);
    }

// ObjectStreamClass.java
    /**
     * Writes non-proxy class descriptor information to given output stream.
     */
    void writeNonProxy(ObjectOutputStream out) throws IOException {
        out.writeUTF(name);
        out.writeLong(getSerialVersionUID());

        byte flags = 0;
        if (externalizable) {
            flags |= ObjectStreamConstants.SC_EXTERNALIZABLE;
            int protocol = out.getProtocolVersion();
            if (protocol != ObjectStreamConstants.PROTOCOL_VERSION_1) {
                flags |= ObjectStreamConstants.SC_BLOCK_DATA;
            }
        } else if (serializable) {
            flags |= ObjectStreamConstants.SC_SERIALIZABLE;
        }
        if (hasWriteObjectData) {
            flags |= ObjectStreamConstants.SC_WRITE_METHOD;
        }
        if (isEnum) {
            flags |= ObjectStreamConstants.SC_ENUM;
        }
        out.writeByte(flags);

        out.writeShort(fields.length);
        for (int i = 0; i < fields.length; i++) {
            ObjectStreamField f = fields[i];
            out.writeByte(f.getTypeCode());
            out.writeUTF(f.getName());
            if (!f.isPrimitive()) {
                out.writeTypeString(f.getTypeString());
            }
        }
    }
```



###### 2.2.2 writeSerailData

写类数据

```java
    private void writeSerialData(Object obj, ObjectStreamClass desc)
        throws IOException
    {
      // 获取要序列化的对象
        ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
        for (int i = 0; i < slots.length; i++) {
            ObjectStreamClass slotDesc = slots[i].desc;
          // 如果对象中重写了writeObject
            if (slotDesc.hasWriteObjectMethod()) {
                PutFieldImpl oldPut = curPut;
                curPut = null;
                SerialCallbackContext oldContext = curContext;

                if (extendedDebugInfo) {
                    debugInfoStack.push(
                        "custom writeObject data (class \"" +
                        slotDesc.getName() + "\")");
                }
                try {
                    curContext = new SerialCallbackContext(obj, slotDesc);
                    bout.setBlockDataMode(true);
                  // 反射调用重写的writeObject
                    slotDesc.invokeWriteObject(obj, this);
                    bout.setBlockDataMode(false);
                    bout.writeByte(TC_ENDBLOCKDATA);
                } finally {
                    curContext.setUsed();
                    curContext = oldContext;
                    if (extendedDebugInfo) {
                        debugInfoStack.pop();
                    }
                }

                curPut = oldPut;
            } else {
              // 默认的序列化方法
                defaultWriteFields(obj, slotDesc);
            }
        }
    }
```

###### 2.2.2.1 defaultWriteFields

真正写类数据的地方

```java
    /**
     * Fetches and writes values of serializable fields of given object to
     * stream.  The given class descriptor specifies which field values to
     * write, and in which order they should be written.
     */
    private void defaultWriteFields(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        Class<?> cl = desc.forClass();
        if (cl != null && obj != null && !cl.isInstance(obj)) {
            throw new ClassCastException();
        }

        desc.checkDefaultSerialize();

      // 获取对象中原生类型的字段的总长度
        int primDataSize = desc.getPrimDataSize();
        if (primDataSize > 0) {
            if (primVals == null || primVals.length < primDataSize) {
                primVals = new byte[primDataSize];
            }
          // 获取对象中原生类型的字段的所有值，放入字节数组primVals
          // *这里是最终写对象里字段的值的地方*
            desc.getPrimFieldValues(obj, primVals);
          // 将primVals写入输出流
            bout.write(primVals, 0, primDataSize, false);
        }

        int numObjFields = desc.getNumObjFields();
        if (numObjFields > 0) {
            ObjectStreamField[] fields = desc.getFields(false);
            Object[] objVals = new Object[numObjFields];
          // 剩下的非原生类型的字段
            int numPrimFields = fields.length - objVals.length;
            desc.getObjFieldValues(obj, objVals);
            for (int i = 0; i < objVals.length; i++) {
                if (extendedDebugInfo) {
                    debugInfoStack.push(
                        "field (class \"" + desc.getName() + "\", name: \"" +
                        fields[numPrimFields + i].getName() + "\", type: \"" +
                        fields[numPrimFields + i].getType() + "\")");
                }
                try {
                  // 递归序列化这个非原生类型字段对象
                    writeObject0(objVals[i],
                                 fields[numPrimFields + i].isUnshared());
                } finally {
                    if (extendedDebugInfo) {
                        debugInfoStack.pop();
                    }
                }
            }
        }
    }
```



到这序列化的主体逻辑就结束了。

反序列化的主要操作和序列化都是一一对应的。读出每个类标志，用对应方式去解析。

```java
    private Object readObject0(boolean unshared) throws IOException {
        boolean oldMode = bin.getBlockDataMode();
        if (oldMode) {
            int remain = bin.currentBlockRemaining();
            if (remain > 0) {
                throw new OptionalDataException(remain);
            } else if (defaultDataEnd) {
                /*
                 * Fix for 4360508: stream is currently at the end of a field
                 * value block written via default serialization; since there
                 * is no terminating TC_ENDBLOCKDATA tag, simulate
                 * end-of-custom-data behavior explicitly.
                 */
                throw new OptionalDataException(true);
            }
            bin.setBlockDataMode(false);
        }

      // 这里开始
        byte tc;
        while ((tc = bin.peekByte()) == TC_RESET) {
            bin.readByte();
            handleReset();
        }

        depth++;
        totalObjectRefs++;
        try {
            switch (tc) {
                case TC_NULL:
                    return readNull();

                // 共享的句柄索引
                case TC_REFERENCE:
                    return readHandle(unshared);

                case TC_CLASS:
                    return readClass(unshared);

                // 类说明
                case TC_CLASSDESC:
                case TC_PROXYCLASSDESC:
                    return readClassDesc(unshared);

                // 字符串
                case TC_STRING:
                case TC_LONGSTRING:
                    return checkResolve(readString(unshared));

                case TC_ARRAY:
                    return checkResolve(readArray(unshared));

                case TC_ENUM:
                    return checkResolve(readEnum(unshared));

                // JavaBean
                case TC_OBJECT:
                    return checkResolve(readOrdinaryObject(unshared));

                case TC_EXCEPTION:
                    IOException ex = readFatalException();
                    throw new WriteAbortedException("writing aborted", ex);

                case TC_BLOCKDATA:
                case TC_BLOCKDATALONG:
                    if (oldMode) {
                        bin.setBlockDataMode(true);
                        bin.peek();             // force header read
                        throw new OptionalDataException(
                            bin.currentBlockRemaining());
                    } else {
                        throw new StreamCorruptedException(
                            "unexpected block data");
                    }

                case TC_ENDBLOCKDATA:
                    if (oldMode) {
                        throw new OptionalDataException(true);
                    } else {
                        throw new StreamCorruptedException(
                            "unexpected end of block data");
                    }

                default:
                    throw new StreamCorruptedException(
                        String.format("invalid type code: %02X", tc));
            }
        } finally {
            depth--;
            bin.setBlockDataMode(oldMode);
        }
    }
```





#### 其他知识点

###### 字段值的读取和写入

ObjectOutputStream写值的逻辑：获取到当前对象中的原生类型字段，primDataSize是对象中所有原生类型字段的总长度，`desc.getPrimFieldValues(obj, primVals);`是将对象中的所有原生字段的值写入primVals这个字节数组，获取对象的值时是用Unsafe类直接通过偏移量取值，`unsafe.getBoolean(obj, offset)`这样的方式。

同理，ObjectInputStream读值的逻辑，`unsafe.putBoolean(obj, key, Bits.getBoolean(buf, off));`

实现在`ObjectStreamClass#getPrimFieldValues`和`ObjectStreamClass#setPrimFieldValues`



###### 共享句柄

序列化过程中出现过的对象、字符串、数值，甚至拼接出来的类信息，如果是shared模式，都不会再完整序列化一次，只会输出handles句柄的索引。

![image-20200621001054494](/github/northernw.github.io/image/image-20200621001054494.png)

写入句柄的地方

![image-20200621001613320](/github/northernw.github.io/image/image-20200621001613320.png)



#### 一个直观的例子

举上面Person的例子。

write Person: person -> write Integer: age & write String: name

write Integer: age -> write int value(11) & write Number : null

write String: name



Person里再加Birthday one, Birthday two属性。

Birthday属性 year, month, day.

```java
@Slf4j
@Data
public class Person implements Serializable {
    private static final long serialVersionUID = 6666716291353949192L;
    private Integer age;
    private String name;
    private Birthday one;
    private Birthday two;

    @Data
    class Birthday implements Serializable {
        private Integer year;
        private Integer month;
        private Integer day;
    }

    @Test
    public void test() throws IOException, ClassNotFoundException {
        Person person = new Person();
        person.setAge(22);
        person.setName("lily");

        Birthday one = new Birthday();
        one.setYear(2020);

        Birthday two = new Birthday();
        two.setYear(2019);
        person.setOne(one);
        person.setTwo(two);
//        person.setIsParent(false);
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("java.person.txt"));
        objectOutputStream.writeObject(person);
        log.info("writeObject = {}", person);

        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("java.person.txt"));
        person = (Person) objectInputStream.readObject();
        log.info("readObject = {}", person);
    }
}
```



以JavaBean为例，不严谨的一份序列化结果接近这样：

魔数、版本号只在初始化写一次

魔数、版本号 | 类标志、类信息开始、类名称、序列化协议版本、SUID、一些序列信息标志（比如是Serializable还是externalizable..）、字段个数、（for每个字段的）字段TypeCode、字段名称、（非原生类型的）字段类型、类信息结束（向上递归父类的信息）【父类的类标志....（非原生类型的）字段类型、类信息结束】（从父类到子类，for每个类）所有原生字段的值+递归所有非原生字段的序列化

~~高亮那一段解释不来...~~可以解释了，Birthday的fields如debug截图

![image-20200621002637015](/github/northernw.github.io/image/image-20200621002637015.png)

![image-20200621003459769](/github/northernw.github.io/image/image-20200621003459769.png)





## Gson

JSON (JavaScript Object Notation)是一种轻量级数据交换格式。

### 使用

依赖 maven

```xml
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.8.6</version>
      <scope>compile</scope>
    </dependency>
```

#### 基础用法

api非常的简单、易用

##### 基本类型

```java
// Serialization
Gson gson = new Gson();
gson.toJson(1);            // ==> 1
gson.toJson("abcd");       // ==> "abcd"
gson.toJson(new Long(10)); // ==> 10
int[] values = { 1 };
gson.toJson(values);       // ==> [1]

// Deserialization
int one = gson.fromJson("1", int.class);
Integer one = gson.fromJson("1", Integer.class);
Long one = gson.fromJson("1", Long.class);
Boolean false = gson.fromJson("false", Boolean.class);
String str = gson.fromJson("\"abc\"", String.class);
String[] anotherStr = gson.fromJson("[\"abc\"]", String[].class);
```

##### 引用类型

```java
class BagOfPrimitives {
  private int value1 = 1;
  private String value2 = "abc";
  private transient int value3 = 3;
  BagOfPrimitives() {
    // no-args constructor
  }
}

// Serialization
BagOfPrimitives obj = new BagOfPrimitives();
Gson gson = new Gson();
String json = gson.toJson(obj);  

// ==> json is {"value1":1,"value2":"abc"}

// Deserialization
BagOfPrimitives obj2 = gson.fromJson(json, BagOfPrimitives.class);
// ==> obj2 is just like obj
```

##### 数组

```java
Gson gson = new Gson();
int[] ints = {1, 2, 3, 4, 5};
String[] strings = {"abc", "def", "ghi"};

// Serialization
gson.toJson(ints);     // ==> [1,2,3,4,5]
gson.toJson(strings);  // ==> ["abc", "def", "ghi"]

// Deserialization
int[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class); 
// ==> ints2 will be same as ints
```

##### 泛型

```java
class Foo<T> {
  T value;
}
Gson gson = new Gson();
Foo<Bar> foo = new Foo<Bar>();
gson.toJson(foo); // May not serialize foo.value correctly

gson.fromJson(json, foo.getClass()); // foo.value 不能反序列化成 Bar

// 正确反序列化泛型的方式 利用TypeToken
Type fooType = new TypeToken<Foo<Bar>>() {}.getType();
gson.toJson(foo, fooType);

gson.fromJson(json, fooType);
```





### 实现原理



#### 核心：TypeAdapter

类型适配器，用到了适配器模式，作为JsonWriter/JsonReader和类型T的对象之间的适配器，将一个对象写入json数据，或从json数据中读入一个对象。

```java
public abstract class TypeAdapter<T> {
  
  public abstract void write(JsonWriter out, T value) throws IOException;
  
  public abstract T read(JsonReader in) throws IOException;
  
  ...
}
```

Gson为每一种类型创建一个TypeAdapter，同样的，每一个Type都对应唯一一个TypeAdapter。

在Gson中，类型Type大致可以分为两类：

1. 基本平台类型，如int.class、Integer.class、String.class、Url.class等
2. 组合及自定义类型，如Collection.class、Map.class和用户自定义的JavaBean等

引用[这里](https://juejin.im/post/5c1473d9e51d4529ee23645f#heading-2)一张示意图：

Gson根据传入的Type找对应的TypeAdapter，如果是基本平台类型，利用TypeAdapter可直接读写json，如果是组合及自定义类型，则在对应的TypeAdapter里封装了对内部属性的处理，是一个迭代的过程（和上面Java自带的序列化writeOrdinaryObject&readOrdinaryObject是很类似的）。

![image-20200622204756920](/github/northernw.github.io/image/image-20200622204756920.png)



#### 源码分析

##### Gson属性和构造函数

一些属性

```java
  /**
   * 避免获取adapter的时候产生无限递归
   */
  private final ThreadLocal<Map<TypeToken<?>, FutureTypeAdapter<?>>> calls
      = new ThreadLocal<Map<TypeToken<?>, FutureTypeAdapter<?>>>();
// 缓存的adapter
  private final Map<TypeToken<?>, TypeAdapter<?>> typeTokenCache = new ConcurrentHashMap<TypeToken<?>, TypeAdapter<?>>();
// 注册的adapter工厂
final List<TypeAdapterFactory> factories;
```

最终调用的构造函数

`new Gson()`得到默认的实例，或者用`GsonBuilder`自定义一些策略

```java
Gson(Excluder excluder, FieldNamingStrategy fieldNamingStrategy,
      Map<Type, InstanceCreator<?>> instanceCreators, boolean serializeNulls,
      boolean complexMapKeySerialization, boolean generateNonExecutableGson, boolean htmlSafe,
      boolean prettyPrinting, boolean lenient, boolean serializeSpecialFloatingPointValues,
      LongSerializationPolicy longSerializationPolicy, String datePattern, int dateStyle,
      int timeStyle, List<TypeAdapterFactory> builderFactories,
      List<TypeAdapterFactory> builderHierarchyFactories,
      List<TypeAdapterFactory> factoriesToBeAdded) {
  // 默认的一些序列化策略，比如字段名称策略，自定义的实例化构造器，pretty输出...
    this.excluder = excluder;
    this.fieldNamingStrategy = fieldNamingStrategy;
    this.instanceCreators = instanceCreators;
    this.constructorConstructor = new ConstructorConstructor(instanceCreators);
    this.serializeNulls = serializeNulls;
    this.complexMapKeySerialization = complexMapKeySerialization;
    this.generateNonExecutableJson = generateNonExecutableGson;
    this.htmlSafe = htmlSafe;
    this.prettyPrinting = prettyPrinting;
    this.lenient = lenient;
    this.serializeSpecialFloatingPointValues = serializeSpecialFloatingPointValues;
    this.longSerializationPolicy = longSerializationPolicy;
    this.datePattern = datePattern;
    this.dateStyle = dateStyle;
    this.timeStyle = timeStyle;
    this.builderFactories = builderFactories;
    this.builderHierarchyFactories = builderHierarchyFactories;

  // 注册adapter工厂
    List<TypeAdapterFactory> factories = new ArrayList<TypeAdapterFactory>();

    // built-in type adapters that cannot be overridden
    factories.add(TypeAdapters.JSON_ELEMENT_FACTORY);
    factories.add(ObjectTypeAdapter.FACTORY);

    // the excluder must precede all adapters that handle user-defined types
    factories.add(excluder);

    // users' type adapters 如果有自定义的工厂，在这里被加入，顺序比较靠前
    factories.addAll(factoriesToBeAdded);

    // type adapters for basic platform types 基本平台类型的adapter工厂
    factories.add(TypeAdapters.STRING_FACTORY);
    factories.add(TypeAdapters.INTEGER_FACTORY);
    factories.add(TypeAdapters.BOOLEAN_FACTORY);
    factories.add(TypeAdapters.BYTE_FACTORY);
    factories.add(TypeAdapters.SHORT_FACTORY);
    TypeAdapter<Number> longAdapter = longAdapter(longSerializationPolicy);
    factories.add(TypeAdapters.newFactory(long.class, Long.class, longAdapter));
    factories.add(TypeAdapters.newFactory(double.class, Double.class,
            doubleAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.newFactory(float.class, Float.class,
            floatAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.NUMBER_FACTORY);
    factories.add(TypeAdapters.ATOMIC_INTEGER_FACTORY);
    factories.add(TypeAdapters.ATOMIC_BOOLEAN_FACTORY);
    factories.add(TypeAdapters.newFactory(AtomicLong.class, atomicLongAdapter(longAdapter)));
    factories.add(TypeAdapters.newFactory(AtomicLongArray.class, atomicLongArrayAdapter(longAdapter)));
    factories.add(TypeAdapters.ATOMIC_INTEGER_ARRAY_FACTORY);
    factories.add(TypeAdapters.CHARACTER_FACTORY);
    factories.add(TypeAdapters.STRING_BUILDER_FACTORY);
    factories.add(TypeAdapters.STRING_BUFFER_FACTORY);
    factories.add(TypeAdapters.newFactory(BigDecimal.class, TypeAdapters.BIG_DECIMAL));
    factories.add(TypeAdapters.newFactory(BigInteger.class, TypeAdapters.BIG_INTEGER));
    factories.add(TypeAdapters.URL_FACTORY);
    factories.add(TypeAdapters.URI_FACTORY);
    factories.add(TypeAdapters.UUID_FACTORY);
    factories.add(TypeAdapters.CURRENCY_FACTORY);
    factories.add(TypeAdapters.LOCALE_FACTORY);
    factories.add(TypeAdapters.INET_ADDRESS_FACTORY);
    factories.add(TypeAdapters.BIT_SET_FACTORY);
    factories.add(DateTypeAdapter.FACTORY);
    factories.add(TypeAdapters.CALENDAR_FACTORY);
    factories.add(TimeTypeAdapter.FACTORY);
    factories.add(SqlDateTypeAdapter.FACTORY);
    factories.add(TypeAdapters.TIMESTAMP_FACTORY);
    factories.add(ArrayTypeAdapter.FACTORY);
    factories.add(TypeAdapters.CLASS_FACTORY);

    // type adapters for composite and user-defined types 复合类型和自定义类型的adapter工厂
    factories.add(new CollectionTypeAdapterFactory(constructorConstructor));
    factories.add(new MapTypeAdapterFactory(constructorConstructor, complexMapKeySerialization));
    this.jsonAdapterFactory = new JsonAdapterAnnotationTypeAdapterFactory(constructorConstructor);
    factories.add(jsonAdapterFactory);
    factories.add(TypeAdapters.ENUM_FACTORY);
    factories.add(new ReflectiveTypeAdapterFactory(
        constructorConstructor, fieldNamingStrategy, excluder, jsonAdapterFactory));

    this.factories = Collections.unmodifiableList(factories);
  }
```





##### toJson

```java
@Slf4j
public class LearningGsonTest {
    @Test
    public void testGson() {
        Person person = new Person();
        person.setAge(22);
        person.setName("lily");
        log.info("person = {}", person);

        Gson gson = new Gson();
        String json = gson.toJson(person);
        log.info("json = {}", json);

        person = gson.fromJson(json, Person.class);
        log.info("person = {}", person);
    }
}
```



```java
  public String toJson(Object src) {
    if (src == null) {
      return toJson(JsonNull.INSTANCE);
    }
    // 取Object的Type
    // Class.class属于Type的一种
    return toJson(src, src.getClass());
  }

  public String toJson(Object src, Type typeOfSrc) {
    StringWriter writer = new StringWriter();
    toJson(src, typeOfSrc, writer);
    return writer.toString();
  }

  public void toJson(Object src, Type typeOfSrc, Appendable writer) throws JsonIOException {
    try {
      JsonWriter jsonWriter = newJsonWriter(Streams.writerForAppendable(writer));
      toJson(src, typeOfSrc, jsonWriter);
    } catch (IOException e) {
      throw new JsonIOException(e);
    }
  }

// 最终都会到这个方法
  public void toJson(Object src, Type typeOfSrc, JsonWriter writer) throws JsonIOException {
    // 关键步骤1，获取到对应的TypeAdapter
    TypeAdapter<?> adapter = getAdapter(TypeToken.get(typeOfSrc));
    boolean oldLenient = writer.isLenient();
    writer.setLenient(true);
    boolean oldHtmlSafe = writer.isHtmlSafe();
    writer.setHtmlSafe(htmlSafe);
    boolean oldSerializeNulls = writer.getSerializeNulls();
    writer.setSerializeNulls(serializeNulls);
    try {
      // 关键步骤2，利用TypeAdapter写出json数据
      ((TypeAdapter<Object>) adapter).write(writer, src);
    } catch (IOException e) {
      throw new JsonIOException(e);
    } catch (AssertionError e) {
      AssertionError error = new AssertionError("AssertionError (GSON " + GsonBuildConfig.VERSION + "): " + e.getMessage());
      error.initCause(e);
      throw error;
    } finally {
      writer.setLenient(oldLenient);
      writer.setHtmlSafe(oldHtmlSafe);
      writer.setSerializeNulls(oldSerializeNulls);
    }
  }
```

看一下`TypeToken.get(typeOfSrc)`

```java
  /**
   * Gets type literal for the given {@code Type} instance.
   */
  public static TypeToken<?> get(Type type) {
    return new TypeToken<Object>(type);
  }

  /**
   * Unsafe. Constructs a type literal manually.
   */
  @SuppressWarnings("unchecked")
  TypeToken(Type type) {
    // 带泛型信息的类型
    this.type = $Gson$Types.canonicalize($Gson$Preconditions.checkNotNull(type));
    // 泛型擦除的类型，比如List.class
    this.rawType = (Class<? super T>) $Gson$Types.getRawType(this.type);
    this.hashCode = this.type.hashCode();
  }
```



穿插一下Java类型（Type）系统的介绍。

详细内容可以看这两篇文章Java中的Type类型详解](https://juejin.im/post/5adefaba518825670e5cb44d)和[Java Type类型](https://www.jianshu.com/p/39cc237ad815)，或者Google下。

简单说明下

1. Type是Java语言中所有类型的公共父接口，有4个扩展接口，主要是用来记录泛型的信息（一个泛型中的信息可能是另一个泛型）
   1. GenericArrayType：泛型数组类型，比如`T[] array`、`List<String>[] array`中的`T[]`、`List<String>[]`
   2. ParameterizedType：参数化类型，比如`Lis<String> list`、`Map<String,Object> map`中的`Lis<String>`、`Map<String,Object>`
   3. TypeVariable：类型变量，比如`List<T> list`中的`T`、`Map<K,V> map`、`E[] array`中的`K`、`V`、`E`（一般外层还有一个GenericArrayType或者ParameterizedType）
   4. WildcardType：通配符类型，比如`Map<? extends A,? super B> map`中的`? extends A`、`? super B`，`A`称为上界，`B`称为下界；如果只有`List<?> list`，默认下界是String，默认上界是Object
2. Class是Type的一个直接实现类
3. 其他4个接口都有自己的实现类，在sun.reflect包下

![image-20200624164140296](/github/northernw.github.io/image/image-20200624164140296.png)

举例子：

比如`List<T extends Map>[] array`是一个泛型数组类型GenericArrayType，其中`List<T>`是ParameterizedType，`T`是TypeVariable

genericComponentType可以理解为数组中每个元素的（泛型）类型

![image-20200624174833587](/github/northernw.github.io/image/image-20200624174833587.png)

再如`List<? extends Map> list`，本身是一个ParameterizedType，`? extends Map`是WildcardType

![image-20200624175245701](/github/northernw.github.io/image/image-20200624175245701.png)

`T[] array`，本身是GenericArrayType，`T`是TypeVariable

![image-20200624180623826](/github/northernw.github.io/image/image-20200624180623826.png)



##### getAdapter

```java
  /**
   * Returns the type adapter for {@code} type.
   *
   * @throws IllegalArgumentException if this GSON cannot serialize and
   *     deserialize {@code type}.
   */
  @SuppressWarnings("unchecked")
  public <T> TypeAdapter<T> getAdapter(TypeToken<T> type) {
    // 先看缓存中有无对应的adapter
    TypeAdapter<?> cached = typeTokenCache.get(type == null ? NULL_KEY_SURROGATE : type);
    if (cached != null) {
      return (TypeAdapter<T>) cached;
    }

    // FutureTypeAdapter的本地缓存，避免无限递归取adapter
    Map<TypeToken<?>, FutureTypeAdapter<?>> threadCalls = calls.get();
    boolean requiresThreadLocalCleanup = false;
    if (threadCalls == null) {
      threadCalls = new HashMap<TypeToken<?>, FutureTypeAdapter<?>>();
      calls.set(threadCalls);
      requiresThreadLocalCleanup = true;
    }

    // the key and value type parameters always agree
    // 如果在FutureTypeAdapter的本地缓存中找到当前类型，表明这个类型的adapter正在创建中
    FutureTypeAdapter<T> ongoingCall = (FutureTypeAdapter<T>) threadCalls.get(type);
    if (ongoingCall != null) {
      return ongoingCall;
    }

    try {
      // 放入future的本地缓存
      FutureTypeAdapter<T> call = new FutureTypeAdapter<T>();
      threadCalls.put(type, call);

      // 遍历adapter工厂，创建adapter
      // 工厂创建的逻辑是，当前类型不是这个工厂负责的，返回null，反之创意一个adapter实例返回，接下来会讲一个普通类型的工厂、一个复合工厂创建实例的过程
      for (TypeAdapterFactory factory : factories) {
        TypeAdapter<T> candidate = factory.create(this, type);
        if (candidate != null) {
          // 如果取到adapter，在FutureTypeAdapter中放入真实adapter
          call.setDelegate(candidate);
          typeTokenCache.put(type, candidate);
          return candidate;
        }
      }
      throw new IllegalArgumentException("GSON (" + GsonBuildConfig.VERSION + ") cannot handle " + type);
    } finally {
      // 从future的本地缓存中移除当前类型
      threadCalls.remove(type);

      if (requiresThreadLocalCleanup) {
        calls.remove();
      }
    }
  }
```

为什么要有个FutureTypeAdapter的本地缓存呢？

先剧透ReflectiveTypeAdapterFactory在创建adapter的时候，会遍历属性取对应的adapte。比如Person里有个Person属性，就会递归取Person的adapter..

```java
@Data
@Slf4j
public class Person {
    private Integer age;
    private String name;
    private Person person;
}
```

FutureTypeAdapter本质上是个委托者，内部引用了真正的adapter，在执行json读写时会用这个真正的adapter进行操作。

```java
  static class FutureTypeAdapter<T> extends TypeAdapter<T> {
    // 真正的adapter
    private TypeAdapter<T> delegate;

    public void setDelegate(TypeAdapter<T> typeAdapter) {
      if (delegate != null) {
        throw new AssertionError();
      }
      delegate = typeAdapter;
    }

    @Override public T read(JsonReader in) throws IOException {
      if (delegate == null) {
        throw new IllegalStateException();
      }
      // 用真正adapter进行读
      return delegate.read(in);
    }

    @Override public void write(JsonWriter out, T value) throws IOException {
      if (delegate == null) {
        throw new IllegalStateException();
      }
      // 用真正adapter进行写
      delegate.write(out, value);
    }
  }
```



##### AdapterFactory

平台基础类型的Adapter是预先定义好的，每个类型对应一个adapter，比如String类型的AdapterFactory返回的adapter永远是`TypeAdapters.STRING`

复合类型和自定义类型的Adapter是需要动态创建的，因为泛型不同、JavaBean的属性不同，等等

###### STRING_FACTORY和'STRING_ADAPTER'

实际上没有STRING_ADAPTER这个名字，真正名字是STRING

```java
// TypeAdapterFactory是一个只有create方法的工厂，用于创建adapter
public interface TypeAdapterFactory {
  <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type);
}

// String类型的工厂
public static final TypeAdapterFactory STRING_FACTORY = newFactory(String.class, STRING); // STRING是个adapter

// 创建一个工厂实例的公共方法...
  public static <TT> TypeAdapterFactory newFactory(
      final Class<TT> type, final TypeAdapter<TT> typeAdapter) {
    return new TypeAdapterFactory() {
      @SuppressWarnings("unchecked") // we use a runtime check to make sure the 'T's equal
      @Override public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> typeToken) {
        // 这一类工厂创建adapter的逻辑是一样的
        // 如果传入的typeToken.getRawType()和预先定义的type是相同的，就返回预先定义的adapter，否则返回null（表示这个rawType不是当前工厂和adapter负责的）
        return typeToken.getRawType() == type ? (TypeAdapter<T>) typeAdapter : null;
      }
      @Override public String toString() {
        return "Factory[type=" + type.getName() + ",adapter=" + typeAdapter + "]";
      }
    };
  }

// String类型的TypeAdapter
  public static final TypeAdapter<String> STRING = new TypeAdapter<String>() {
    @Override
    public String read(JsonReader in) throws IOException {
      JsonToken peek = in.peek();
      // 如果是null
      if (peek == JsonToken.NULL) {
        in.nextNull();
        return null;
      }
      // 某个兼容..先跳过
      /* coerce booleans to strings for backwards compatibility */
      if (peek == JsonToken.BOOLEAN) {
        return Boolean.toString(in.nextBoolean());
      }
      // 调用JsonReader读json
      return in.nextString();
    }
    @Override
    public void write(JsonWriter out, String value) throws IOException {
      // 调用JsonWriter写json
      out.value(value);
    }
  };
```

Integer、Long、Boolean、AtomicInteger等等平台基础类型的factory的框架是类似的，或者一样的，有细微的差别（比如同时支持非包装类型和包装类型）



###### CollectionTypeAdapterFactory

集合类型的factory和adapter

`MapTypeAdapterFactory`和集合的很类似

```java
public final class CollectionTypeAdapterFactory implements TypeAdapterFactory {
  private final ConstructorConstructor constructorConstructor;

  public CollectionTypeAdapterFactory(ConstructorConstructor constructorConstructor) {
    this.constructorConstructor = constructorConstructor;
  }

  // create在这里
  @Override
  public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> typeToken) {
    Type type = typeToken.getType();

    // 如果Collection不是rawType的超类/超接口，那就不是集合工厂来负责
    Class<? super T> rawType = typeToken.getRawType();
    if (!Collection.class.isAssignableFrom(rawType)) {
      return null;
    }

    // 获取集合元素的类型，这里是带泛型信息的类型
    Type elementType = $Gson$Types.getCollectionElementType(type, rawType);
    // 取元素类型的adapter
    TypeAdapter<?> elementTypeAdapter = gson.getAdapter(TypeToken.get(elementType));
    // 获取构造器，如果没有自定义等等情况，一般是用反射的constructor
    ObjectConstructor<T> constructor = constructorConstructor.get(typeToken);

    @SuppressWarnings({"unchecked", "rawtypes"}) // create() doesn't define a type parameter
    // 创建一个集合类型的adapter实例
    TypeAdapter<T> result = new Adapter(gson, elementType, elementTypeAdapter, constructor);
    return result;
  }

  private static final class Adapter<E> extends TypeAdapter<Collection<E>> {
    // 集合元素的adapter
    private final TypeAdapter<E> elementTypeAdapter;
    // 构造器
    private final ObjectConstructor<? extends Collection<E>> constructor;

    public Adapter(Gson context, Type elementType,
        TypeAdapter<E> elementTypeAdapter,
        ObjectConstructor<? extends Collection<E>> constructor) {
      // 这里元素adapter还有一个封装，会在writeJson时取运行时的Type，毕竟运行时的最准确（比如定义时候是List<? extends Cat>，运行时实际放入的是Cat接口的一个实现WhiteCat），再根据运行时Type取相应adapter...
      this.elementTypeAdapter =
          new TypeAdapterRuntimeTypeWrapper<E>(context, elementTypeAdapter, elementType);
      this.constructor = constructor;
    }

    // 读json
    @Override public Collection<E> read(JsonReader in) throws IOException {
      if (in.peek() == JsonToken.NULL) {
        in.nextNull();
        return null;
      }

      Collection<E> collection = constructor.construct();
      // 读入 [
      in.beginArray();
      while (in.hasNext()) {
        // 委托元素adapter读每个元素
        E instance = elementTypeAdapter.read(in);
        // 元素放入集合
        collection.add(instance);
      }
      // 读入 ]
      in.endArray();
      return collection;
    }

    @Override public void write(JsonWriter out, Collection<E> collection) throws IOException {
      if (collection == null) {
        out.nullValue();
        return;
      }

      // 写 ]
      out.beginArray();
      for (E element : collection) {
        // 写每个元素
        elementTypeAdapter.write(out, element);
      }
      // 写 ]
      out.endArray();
    }
  }
}
```



TypeAdapterRuntimeTypeWrapper

```java
  @Override
  public void write(JsonWriter out, T value) throws IOException {
    // 优先级 运行时类型adapter（最高） > 声明类型adapter > 运行时类型reflective adapter > 声明类型reflective adapter
    // Order of preference for choosing type adapters
    // First preference: a type adapter registered for the runtime type
    // Second preference: a type adapter registered for the declared type
    // Third preference: reflective type adapter for the runtime type (if it is a sub class of the declared type)
    // Fourth preference: reflective type adapter for the declared type

    TypeAdapter chosen = delegate;
    // 运行时Type
    Type runtimeType = getRuntimeTypeIfMoreSpecific(type, value);
    // 如果runtimeType和传入的Type（即用户声明的对象的Type）不一样
    if (runtimeType != type) {
      TypeAdapter runtimeTypeAdapter = context.getAdapter(TypeToken.get(runtimeType));
      if (!(runtimeTypeAdapter instanceof ReflectiveTypeAdapterFactory.Adapter)) {
        // The user registered a type adapter for the runtime type, so we will use that
        // ReflectiveTypeAdapterFactory是注册的最后一个factory
        // 如果运行时TypeAdapter不是最后一种Adapter
        chosen = runtimeTypeAdapter;
      } else if (!(delegate instanceof ReflectiveTypeAdapterFactory.Adapter)) {
        // The user registered a type adapter for Base class, so we prefer it over the
        // reflective type adapter for the runtime type
        // （delegate是根据用户声明的类型找到的）如果delegate也不是最后一种，用delegate
        chosen = delegate;
      } else {
        // Use the type adapter for runtime type
        chosen = runtimeTypeAdapter;
      }
    }
    chosen.write(out, value);
  }
```



https://juejin.im/post/5c1473d9e51d4529ee23645f#heading-8



### 小结

## Jackson

## FastJson 



## 其他序列化



# 性能对比





# 参考

1. [Java序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf#heading-8)

2. [Java 序列化和反序列化的几篇文章](https://www.cnblogs.com/binarylei/category/1159503.html)
3. [Gson源码解析和它的设计模式](https://juejin.im/post/5c1473d9e51d4529ee23645f)


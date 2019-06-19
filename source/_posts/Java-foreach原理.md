---
title: Java foreach原理
date: 2019-06-19 16:34:16
tags:
---
从字节码可以看出，foreach使用了Iterator迭代器，循环判断hashNext()，用next()取操作对象。
Collection接口继承了Iterable接口，可以获取iterator对象。【Iterator<T> iterator();】
集合对象有对应的xxIterator，实现具体的hashNext()、next()、remove()等操作。
反编译源码更直观。


测试源码
```
public class ForEachTest {

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        for (Integer i : list) {
            System.out.println(i);
        }
    }
}

```

IDE反编译源码
```
public class ForEachTest {
    public ForEachTest() {
    }

    public static void main(String[] args) {
        List<Integer> list = new ArrayList();
        list.add(1);
        list.add(2);
        Iterator var2 = list.iterator();

        while(var2.hasNext()) {
            Integer i = (Integer)var2.next();
            System.out.println(i);
        }

    }
}
```

字节码foreach部分节选

```
    for (Integer i : list) {
        System.out.println(i);
    }

   L3
    LINENUMBER 17 L3
    ALOAD 1
    INVOKEINTERFACE java/util/List.iterator ()Ljava/util/Iterator; (itf)
    ASTORE 2
   L4
   FRAME APPEND [java/util/List java/util/Iterator]
    ALOAD 2
    INVOKEINTERFACE java/util/Iterator.hasNext ()Z (itf)
    IFEQ L5
    ALOAD 2
    INVOKEINTERFACE java/util/Iterator.next ()Ljava/lang/Object; (itf)
    CHECKCAST java/lang/Integer
    ASTORE 3
   L6
    LINENUMBER 18 L6
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 3
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/Object;)V
   L7
    LINENUMBER 19 L7
    GOTO L4
```

---
title: foreach中不宜进行remove/add等操作
date: 2019-06-19 17:26:18
tags:
---
## 原因
对比源码和反编译可以看出，都使用iterator进行迭代，差别在于foreach用list.remove(i)，iterator用iterator.remove()。
执行在(Integer)iterator.next()抛出并发修改异常（见反编译代码），原因在于，next()中校验了Itr的expectedModCount和ArrayList的modCount需相等。
list.remove(i)使list的modCount++，而iterator中的expectedModCount不变，由此产生差异。
iterator.remove()会将list的modCount赋值给expectedModCount，无差异。

源码
```
    public static void main(String[] args) {
        List<Integer> list = Lists.newArrayList(1, 2, 3, 4);

        for (Integer i : list) {
            if (i.equals(1)) {
                list.remove(i);
            }
        }

        Iterator<Integer> iterator = list.iterator();

        while (iterator.hasNext()) {
            Integer i = iterator.next();
            if (i.equals(3)) {
                iterator.remove();
            }
        }
    }
```
执行异常信息
```
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:907)
	at java.util.ArrayList$Itr.next(ArrayList.java:857)
	at com.jzt.service.joint.agent.ForEachTest.main(ForEachTest.java:18)
```

IDE反编译
```
    public static void main(String[] args) {
        List<Integer> list = Lists.newArrayList(new Integer[]{1, 2, 3, 4});
        Iterator iterator = list.iterator();

        Integer i;
        while(iterator.hasNext()) {
            i = (Integer)iterator.next();
            if (i.equals(1)) {
                list.remove(i);
            }
        }

        iterator = list.iterator();

        while(iterator.hasNext()) {
            i = (Integer)iterator.next();
            if (i.equals(3)) {
                iterator.remove();
            }
        }

    }
```

附ArrayList中Itr源码节选
```
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
		
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

## 删除倒数第二个元素不抛异常
修改代码为remove 3，在迭代中输出元素，结果如下，foreach不会输出最后一个元素。
list.remove(i)之后的hasNext()返回false，说明iterator的cursor与list的size相等。
iterator.next()后，it.cursor = i + 1 = 3，it.lastRet = 2，指向元素3.
- list.remove((Integer)3)后，list.size = 3。
- iterator.remove()执行的是remove(lastRet)，list.size = 3，修改cursor = lastRet = 2，指向元素4.


输出结果
```
1
2
3

1
2
3
4
```


hasNext源码
```
        int cursor;       // index of next element to return 下一个next返回
        int lastRet = -1; // index of last element returned; -1 if no such 上一个next返回
		
        public boolean hasNext() {
            return cursor != size;
        }
```
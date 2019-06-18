---
title: ArrayList笔记
date: 2019-06-18 15:05:38
tags:
---

## Q&A
1. 为什么for中不能remove，而iterator时可以？
2. 迭代是怎么实现的？

## 关联知识
1. forEach(Consumer<? super E> action) action.accept(elementData[i]) 了解下Consumer，lambda
2. removeIf(Predicate<? super E> filter) if (filter.test(element))


## 要点

### 增
还有add(int,E),addAll(collection<>)等
重点是检查容量和扩容ensureCapacityInternal，在指定位置上赋值新元素

```
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

```

### 删
1. public E remove(int index) rangeCheck(index); 移动后续元素 最后元素赋值释放引用
2. public boolean remove(Object o) null和非null分开处理 fastRemove（与remove不同的是，无需rangeCheck）

### 改
1. public E set(int index, E element) rangeCheck;赋值；return old data

### 查
1. public E get(int index) rangeCheck;return elementData(index);
```
    E elementData(int index) {
        return (E) elementData[index];
    }
```

## 一些源码的解惑
### batchRemove

```
private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                // 这里complement也很巧妙，removeAll把contains false的元素保留，retainAll把contains true的元素保留
                if (c.contains(elementData[r]) == complement) 
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            // 此处是指contains抛出异常，r != size，需要把size-r个（没有被处理）的元素拷贝到w位置后
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }


```


### removeIf
用了BitSet，filter test为真时，将该位BitSet设置为true

```
public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            // filter test为真时，将该位BitSet设置为true
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                // 将值为false的元素保留（前移）
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            // 清除多余的空间，赋值null，去掉对对象的引用
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }

```


### replaceAll
UnaryOperator 一元操作

```
List<Integer> list = new ArrayList<Integer>();
for (int i = 0; i < 10; i++) {
    list.add(i);
}
list.replaceAll(x -> x + 10);

```

```
    /**
     * Replaces each element of this list with the result of applying the
     * operator to that element.  Errors or runtime exceptions thrown by
     * the operator are relayed to the caller.
     * @implSpec
     * The default implementation is equivalent to, for this {@code list}:
     * <pre>{@code
     *     final ListIterator<E> li = list.listIterator();
     *     while (li.hasNext()) {
     *         li.set(operator.apply(li.next()));
     *     }
     * }</pre>
     */
    @Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

```


## 经常用到
1. System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
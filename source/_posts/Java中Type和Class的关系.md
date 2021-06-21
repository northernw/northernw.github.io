---
title: Java中Type和Class的关系
tags:
  - null
categories:
  - null
date: 2021-06-15 14:53:32
---



1. Class是Type的一个实现类
2. ParameteriedType 参数化类型（和TypeVariable比起来，ParameteriedType像实参，TypeVariable像形参）
   1. `getRawType():Type` 返回原始类型
   2. `getActualTypeAuguments():Type[]` 返回实参类型
   3. `getOwnerType():Type` 返回所在类的类型
3. TypeVariable 类型变量
   1. `getBounds():Type[]`  返回类型变量的上界类型
   2. `getGenericDeclaration():Type` 返回类型变量所在类的类型（类似于ParameteriedType的~~RawType~~ OwnerType？？感觉这里RawType和OwnerType是等价的，所属类就是它的原始类型）
4. GenericArrayType 泛型数组类型
   1. `getGenericComponentType:Type` 返回数组元素的类型
5. WildcardType：通配符类型
   1. `getLowerBounds():Type[]` 返回下界
   2. `getUpperBounds():Type[]`返回上界
   3. 如果没有上界，默认为Object，没有指定下界，默认为String
   4. extends后面是上界，super后面是下界



```java
List<? extends Number>[] array;
```

1. 整个是GenericArrayType，getGenericComponentType返回 `List<? extends Number>`
2. `List<? extends Number>`是个ParameteriidType，getActualTypeAuguments返回`? extends Number`
3. `? extends Number `是个WildcardType
4. 另外，TypeVariable一般在定义泛型类的时候有，比如List.class就会有TypeVariable


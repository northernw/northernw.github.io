---
title: Gson源码笔记
tags:
  - Gson
categories:
  - 源码解析
  - 设计模式
date: 2019-07-18 20:00:15
---
## Gson源码版本
```java
	<dependency>
		<groupId>com.google.code.gson</groupId>
		<artifactId>gson</artifactId>
		<version>2.4</version>
	</dependency>
```

## summary
- 抽象类TypeAdapter，委托模式，处理json字符串和特定类型对象之间的相互转换。抽象方法read和write在具体的TypeAdapter有各自的实现，比较复杂的包括集合类型的CollectionTypeAdapterFactory.Adapter（比如read时，要实例化集合，再循环处理集合内元素）、自定义对象类型的ReflectiveTypeAdapterFactory.Adapter（比如需要循环处理类内field的read和write）。
- 接口TypeAdapterFactory，抽象工厂模式，创建给定类型的TypeAdapter。基础类型已经预先创建typeAdapter（调用factory.create时，如果是给定的类型，返回预定义的adapter），Collection、自定义类型等包含泛型的，在运行时create（因为会有不同的要素，比如constructor、elementType、boundFields等）。
- 序列化时，泛型类型通过TypeAdapterRuntimeTypeWrapper进行处理。会判断使用rawType带过来的类型还是运行时真实value的类型进行后续处理。
- 责任链模式（？待确认）。在对顶层类型getAdapter过程中，会递归对下层类型进行getAdapter，并保存在上层adapter中；在顶层adapter.write过程中，也递归调用到子类型的adapter.write。ps在基本类型adapter中，才调用JsonReader/Writer读写。


## 设计模式
### 工厂模式
TypeAdapterFactory.create提供给定TypeToken<T>的TypeAdapter<T>.

```java
public interface TypeAdapterFactory {

  /**
   * Returns a type adapter for {@code type}, or null if this factory doesn't
   * support {@code type}.
   */
  <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type);
}

```

## factory和adapter

### Gson内置factory

```java
    List<TypeAdapterFactory> factories = new ArrayList<TypeAdapterFactory>();

	// Gson的类型
    // built-in type adapters that cannot be overridden
    factories.add(TypeAdapters.JSON_ELEMENT_FACTORY);
    factories.add(ObjectTypeAdapter.FACTORY);

    // the excluder must precede all adapters that handle user-defined types
    factories.add(excluder);

	// 用户定义的TypeAdapters
    // user's type adapters
    factories.addAll(typeAdapterFactories);

	// 基础类型
    // type adapters for basic platform types
    factories.add(TypeAdapters.STRING_FACTORY);
    factories.add(TypeAdapters.INTEGER_FACTORY);
    factories.add(TypeAdapters.BOOLEAN_FACTORY);
    factories.add(TypeAdapters.BYTE_FACTORY);
    factories.add(TypeAdapters.SHORT_FACTORY);
    factories.add(TypeAdapters.newFactory(long.class, Long.class,
            longAdapter(longSerializationPolicy)));
    factories.add(TypeAdapters.newFactory(double.class, Double.class,
            doubleAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.newFactory(float.class, Float.class,
            floatAdapter(serializeSpecialFloatingPointValues)));
    factories.add(TypeAdapters.NUMBER_FACTORY);
    factories.add(TypeAdapters.CHARACTER_FACTORY);
    factories.add(TypeAdapters.STRING_BUILDER_FACTORY);
    factories.add(TypeAdapters.STRING_BUFFER_FACTORY);
    factories.add(TypeAdapters.newFactory(BigDecimal.class, TypeAdapters.BIG_DECIMAL));
    factories.add(TypeAdapters.newFactory(BigInteger.class, TypeAdapters.BIG_INTEGER));
    factories.add(TypeAdapters.URL_FACTORY);
    factories.add(TypeAdapters.URI_FACTORY);
    factories.add(TypeAdapters.UUID_FACTORY);
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

	// 组合类型和自定义类型
    // type adapters for composite and user-defined types
    factories.add(new CollectionTypeAdapterFactory(constructorConstructor));
    factories.add(new MapTypeAdapterFactory(constructorConstructor, complexMapKeySerialization));
    factories.add(new JsonAdapterAnnotationTypeAdapterFactory(constructorConstructor));
    factories.add(TypeAdapters.ENUM_FACTORY);
    factories.add(new ReflectiveTypeAdapterFactory(
        constructorConstructor, fieldNamingPolicy, excluder));

```

## TypeToken
匿名内部类有两种语法格式
- new 接口(){}
- new 父类构造器(参数列表){}
TypeToken为第二种
`new TypeToken<List<TwoGeneric<Integer,User>>>(){};`得到的是`TypeToken<List<TwoGeneric<Integer,User>>>`的匿名子类。

## todo
- map 复杂key的序列化
- map peek == JsonToken.BEGIN_ARRAY
- reflectiveTypeAdapter 多个泛型 [done. read时根据name和TypeToken，write时根据TypeAdapterRuntimeTypeWrapper]
- toJson时候，type里带实例的runtime type? [done. TypeAdapterRuntimeTypeWrapper处理]
- JsonAdapterAnnotationTypeAdapterFactory [almost done. 获取filed上注解的TypeAdapter进行后续处理]
- $Gson$Types.resolve [done]
- JsonWriter和JsonReader的读写 [almost done. 写比较简单，不同类型输出；读在fillBuffer时先读到缓存中，不同类型在缓存中操作。]
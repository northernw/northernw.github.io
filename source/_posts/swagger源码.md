---
title: swagger源码笔记
tags:
  - swagger
categories:
  - 源码解析
date: 2019-09-16 15:17:01


---

配置swagger可生成controller的接口文档，比较好奇example value是如何生成的，来一探究竟。

从UI方面看，主要信息来自于definitions，给出了出入参数（汇总为model）的属性，引用了已存在实体。有了type和相关信息，既可以mock数据，也可以实例化对象。

初步看源码，Model和ModelProperty中的type为ResolvedType，是jackson的实现（还没看过jackson源码，哭泣）。



## 总结



### 调用过程

`@EnableSwagger2`注解引入了swagger的各类plugin，在spring容器finish之后，开始这些plugin的工作。在swagger中就是`DocumentationPluginsBootstrapper`。

接下去以Documentation-ApiListing(method)-ApiModel(parameter/OperationModel)等维度，生成文档数据。

主要看了model的生成，ApiDescription及Operation的组织形式没有关注，下次可在启动完成后，看下根documentation的内部结构。

Ps:一个docket看做一个文档，可设置是否启用，group分组，默认分组为default。

![image-20190916203934721](/Users/wangyuanqing1/github/northernw.github.io/image/image-20190916203934721.png)





## 疑问

### ResolvedType在哪里生成？



包括这些依赖

```java
DefaultModelDependencyProvider.java
line: 177

  private List<ResolvedType> resolvedDependencies(ModelContext modelContext) {
    ResolvedType resolvedType = modelContext.alternateFor(modelContext.resolvedType(typeResolver));
    if (isBaseType(ModelContext.fromParent(modelContext, resolvedType))) {
      LOG.debug("Marking base type {} as seen", resolvedType.getSignature());
      modelContext.seen(resolvedType);
      return newArrayList();
    }
  
  // 依赖
    List<ResolvedType> dependencies = newArrayList(resolvedTypeParameters(modelContext, resolvedType));
    dependencies.addAll(resolvedArrayElementType(modelContext, resolvedType));
    dependencies.addAll(resolvedMapType(modelContext, resolvedType));
  // 大部分是在这里产生的最终ResolvedType，其他几个resolve会调用到当前方法resolvedDependencies
    dependencies.addAll(resolvedPropertiesAndFields(modelContext, resolvedType));
    dependencies.addAll(resolvedSubclasses(resolvedType));
    return dependencies;
  }

  private List<ResolvedType> resolvedPropertiesAndFields(ModelContext modelContext, ResolvedType resolvedType) {
    if (modelContext.hasSeenBefore(resolvedType) || enumTypeDeterminer.isEnum(resolvedType.getErasedType())) {
      return newArrayList();
    }
    modelContext.seen(resolvedType);
    List<ResolvedType> properties = newArrayList();
    for (ModelProperty property : nonTrivialProperties(modelContext, resolvedType)) {
      LOG.debug("Adding type {} for parameter {}", property.getType().getSignature(), property.getName());
      if (!isMapType(property.getType())) {
        properties.add(property.getType());
      }
      // 集合类的元素类型
      properties.addAll(maybeFromCollectionElementType(modelContext, property));
      // map的value类型
      properties.addAll(maybeFromMapValueType(modelContext, property));
      // 本身
      properties.addAll(maybeFromRegularType(modelContext, property));
    }
    return properties;
  }
```

![33image-20190916180323867](/Users/wangyuanqing1/github/northernw.github.io/image/image-20190916180323867.png)





### 生成model

看调用栈

* ApiDocumentationScanner

* ApiListingScanner

* OperationModelsProvider

  * ```java
    collectFromReturnType(context); // 出参
    collectParameters(context); // 入参
    collectGlobalModels(context);
    ```

![image-20190916173013716](/Users/wangyuanqing1/github/northernw.github.io/image/swagger-start-model-1.png)





这里解析每个属性的类型，jackson的实现

![image-20190916174254264](/Users/wangyuanqing1/github/northernw.github.io/image/image-20190916174254264.png)



这里记录下，82个typeResolver（在哪里注入的？）

![image-20190916174027934](/Users/wangyuanqing1/github/northernw.github.io/image/image-20190916174027934.png)



```java
// 集合类型 数组类型 --> 容器类型
public static boolean isContainerType(ResolvedType type) {
  return List.class.isAssignableFrom(type.getErasedType()) ||
      Set.class.isAssignableFrom(type.getErasedType()) ||
      (Collection.class.isAssignableFrom(type.getErasedType()) && !Maps.isMapType(type)) ||
      type.isArray();
}
```



## 验证

### 需指定泛型

```java
@ToString
@Getter
@Setter
public class ApiResponse<T> {
    private Integer code;
    private String msg;
    private Boolean success;
    private T data;

    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> response = new ApiResponse<T>();
        response.setData(data);
        response.setCode(1);
        response.setSuccess(Boolean.TRUE);
        return response;
    }
}

@RestController
public class UserController {

    @GetMapping("/list")
    public ApiResponse<List<User>> list(@RequestBody User user) {
        return ApiResponse.success(Lists.newArrayList());
    }

    @GetMapping("/list/no/type")
    public ApiResponse listNoType(@RequestBody User user) {
        return ApiResponse.success(Lists.newArrayList());
    }
}




```

![image-20190916152524962](/Users/wangyuanqing1/github/northernw.github.io/image/swagger-pic.png)



![image-20190916154320343](/Users/wangyuanqing1/github/northernw.github.io/image/swagger-generic-type.png)
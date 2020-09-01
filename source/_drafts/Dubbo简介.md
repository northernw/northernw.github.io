---
title: Dubbo简介
tags:
---



# Introduction to Dubbo

## 概述

根据[官网](http://dubbo.apache.org/zh-cn/)介绍，Dubbo是一款高性能、轻量级的开源Java RPC框架。

提供三大核心功能：

1. 面向接口的远程方法调用
2. 智能容错和负载均衡
3. 服务自动注册和发现



ps：三大功能的大白话版实现原理

1. 面向接口的远程方法调用——为接口生成一个代理实现，完成对接口的透明RPC调用
2. 智能容错和负载均衡——提供了不同的容错策略和负载均衡策略，以Filter的方式在客户端起作用
3. 服务自动注册和发现——注册中心，服务端注册服务（1. 利用zk的心跳，能感知服务端异常，2. 服务端主动注销），客户端拉取服务&监听变化



TODO：

1. 客户端的心跳是做什么用的？



## 使用

接口

```java
public interface DemoService {
    String sayHello(String name);
}
```

实现

```java
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```



### 方式1：集成Spring

spring加载dubbo的配置，基于 [Spring 的 Schema 扩展](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/xsd-configuration.html) 进行加载（解析自定义标签）。

dubbo-demo-provider.xml

```xml
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />

```

服务端启动类

```java
public class Provider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
        System.in.read(); // 按任意键退出
    }
}
```



dubbo-demo-consumer.xml

```xml
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用zookeeper注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />

```

客户端启动

```java
public class Consumer {
    public static void main(String[] args) throws Exception {
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF/spring/dubbo-demo-consumer.xml"});
        DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法
        System.out.println( hello ); // 显示调用结果
    }
}
```





### 方式2：API

provider

```java
public class ApiProvider {
    public static void main(String[] args) {
      // 配置一个服务
        ServiceConfig<HelloService> service = new ServiceConfig<>();
      // 配置接口名称
        service.setInterface(HelloService.class);
      // 配置对实现类的引用
        service.setRef(new HelloServiceImpl());
      // 设置ID
        service.setId("helloService");

        DubboBootstrap bootstrap = DubboBootstrap.getInstance();
        bootstrap.application(new ApplicationConfig("dubbo-demo-api-myprovider"))
          // 注册中心
                .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
          // 配置服务
                .service(service)
          // 启动
                .start()
                .await();
    }
}
```



consumer

```java
public class ApiConsumer {
    public static void main(String[] args) {
      // 引用一个服务
        ReferenceConfig<HelloService> reference = new ReferenceConfig<>();
        reference.setInterface(HelloService.class);

        DubboBootstrap bootstrap = DubboBootstrap.getInstance();
        bootstrap.application(new ApplicationConfig("dubbo-demo-api-myconsumer"))
                .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
                .reference(reference)
                .start();

        HelloService demoService = reference.get();
        String message = demoService.sayHello("world");
        System.out.println(message);
    }
}
```





## 原理介绍

角色图和介绍

框架图和介绍——压缩版也介绍一下，和rpc功能相对应的

###### RPC是什么？





难点

1. SPI机制，原理比较好理解，实现类是javasisst字节码生成的，不是很方便debug跟踪。可以先跳过实现，理解原理为先。
2. 分层比较多，概念有不太统一的地方，会增加理解的难度

### 初始化过程

服务端=服务提供方

客户端=服务调用方

#### 服务端

#### 客户端

### 调用过程

#### 客户端

#### 服务端



###### 智能容错和负载均衡

1. route 路由
2. load balance 负载均衡，默认Random
3. 降级：服务降级——管理端配置，改url，或者写在服务端配置文件里；客户端mock——xxMock服务
4. 容错 failover失败重试


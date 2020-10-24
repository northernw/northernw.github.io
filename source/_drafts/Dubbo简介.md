---
title: Dubbo简介
tags:
---



# Introduction to Dubbo

内容来源于[官网](http://dubbo.apache.org/zh-cn/)

## 概述

根据[官网](http://dubbo.apache.org/zh-cn/)介绍，Dubbo是一款高性能、轻量级的开源Java RPC框架。

提供三大核心功能：

1. 面向接口的远程方法调用
2. 智能容错和负载均衡
3. 服务自动注册和发现



ps：三大功能的大白话版实现原理

1. 面向接口的远程方法调用——为客户端接口生成一个代理实现，完成对接口的透明RPC调用
2. 智能容错和负载均衡——提供了不同的容错策略和负载均衡策略，以Filter的方式在客户端起作用
3. 服务自动注册和发现——注册中心，服务端注册服务（1. 利用zk的心跳，能感知服务端异常，2. 服务端主动注销），客户端拉取服务&监听变化





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
      // 发布一个服务
        ServiceConfig<DemoService> service = new ServiceConfig<>();
      // 配置接口名称
        service.setInterface(DemoService.class);
      // 配置对实现类的引用
        service.setRef(new DemoServiceImpl());
      // 设置ID
        service.setId("DemoService");

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
        ReferenceConfig<DemoService> reference = new ReferenceConfig<>();
        reference.setInterface(DemoService.class);

        DubboBootstrap bootstrap = DubboBootstrap.getInstance();
        bootstrap.application(new ApplicationConfig("dubbo-demo-api-myconsumer"))
                .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
                .reference(reference)
                .start();

        DemoService demoService = reference.get();
        String message = demoService.sayHello("world");
        System.out.println(message);
    }
}
```





## 原理介绍

### RPC是什么？

RPC指远程过程调用【是一个计算机通信协议，有不同的实现方式】。

两台服务器A、B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，因为不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义、传达调用的数据。

调用过程：

![img](/github/northernw.github.io/image/45366c44f775abfd0ac3b43bccc1abc3_1440w.jpg)

1. 服务消费方（client）以调用本地方法的方式调用服务；
2. client stub接收到调用后将方法、参数等组装成能够进行网络传输的消息体；
3. client stub找到服务地址，将消息发送到服务端；
4. server stub收到消息后进行解码；
5. server stub根据解码结果调用本地的服务；
6. 本地服务执行并将结果返回给server stub；
7. server stub将返回结果打包成消息；
8. server stub将消息发送至消费方；
9. client stub接收到消息，进行解码；
10. 服务消费方得到最终结果。



总结起来，rpc要做以下几件事情：

1. 解决通信问题。在客户端和服务端之间建立TCP连接，调用过程的数据都在这个连接里传输。连接可以按需连接，调用结束后就断掉，也可以是长连接，多个过程调用共享同一个连接。
2. 解决寻址问题。客户端需要知道服务端的地址和端口。最直接的实现是手工指定IP和端口。
3. 解决序列化和反序列化问题，将方法的参数、方法的返回值由内存形式序列化为二进制形式，通过网络传输到达另一端，另一端反序列化为内存形式。



更多关于RPC的介绍：

https://www.zhihu.com/question/25536695/answer/36197244

https://www.cs.rutgers.edu/~pxk/417/notes/03-rpc.html



### Dubbo设计

Dubbo做的事情，首先是实现了上面提到的rpc的基础要求（通信、寻址（无注册中心的话，就是消费者直连提供者）、序列化），其次是引入容错&负载均衡和注册中心（~实现自动寻址）。对应Dubbo的三大核心功能。

简单说明下，容错和负载均衡在RPC层面之上：

1. 容错解决的是，调用出错了客户端怎么处理？Dubbo提供了不同的容错策略：比如failfast快速失败，调用出错了直接返回这个出错；比如failover故障切换，默认3次循环调用，即调用成功跳出循环，失败就换一个服务方再尝试。
2. 负载均衡解决的是，有多个服务方，客户端选择哪个进行调用。也有不同的负载均衡策略：比如Random随机，随机选择一个服务方；比如RoundRobin轮询，如果几个服务方权重相同，等价于按顺序取调用不同服务方。

注册中心实现服务自动注册和发现（发现也包含感知异常服务），解决寻址问题。



#### 架构

这个图比较常见了，说明了<u>节点角色和其中的调用关系</u>。

![dubbo-architucture](/github/northernw.github.io/image/dubbo-architecture.jpg)

节点角色说明：

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

调用关系说明：

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

更多内容见[官网说明](http://dubbo.apache.org/zh-cn/docs/user/preface/architecture.html)



#### 整体设计

这是一张很复杂的图..需要有更多认识后再返回来看。先把官网的一些说明都贴出来。这一部分可以先看个图有个印象。

【内容都来自[官网](http://dubbo.apache.org/zh-cn/docs/dev/design.html)】

图例说明：

- 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。
- 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
- 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。
- 图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

![dubbo-framework.jpg (900×674)](/github/northernw.github.io/image/dubbo-framework.jpg)

各层说明：

- config 配置层：对外配置接口，以 `ServiceConfig`, `ReferenceConfig` 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类
- proxy 服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 `ServiceProxy` 为中心，扩展接口为 `ProxyFactory`
- registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 `RegistryFactory`, `Registry`, `RegistryService`
- cluster 路由层：封装多个提供者的路由及负载均衡，并桥接注册中心，以 `Invoker` 为中心，扩展接口为 `Cluster`, `Directory`, `Router`, `LoadBalance`
- monitor 监控层：RPC 调用次数和调用时间监控，以 `Statistics` 为中心，扩展接口为 `MonitorFactory`, `Monitor`, `MonitorService`
- protocol 远程调用层：封装 RPC 调用，以 `Invocation`, `Result` 为中心，扩展接口为 `Protocol`, `Invoker`, `Exporter`
- exchange 信息交换层：封装请求响应模式，同步转异步，以 `Request`, `Response` 为中心，扩展接口为 `Exchanger`, `ExchangeChannel`, `ExchangeClient`, `ExchangeServer`
- transport 网络传输层：抽象 mina 和 netty 为统一接口，以 `Message` 为中心，扩展接口为 `Channel`, `Transporter`, `Client`, `Server`, `Codec`
- serialize 数据序列化层：可复用的一些工具，扩展接口为 `Serialization`, `ObjectInput`, `ObjectOutput`, `ThreadPool`

关系说明【这部分说明对理解系统设计比较有帮助】

- 在 RPC 中，Protocol 是核心层，**也就是只要有 Protocol + Invoker + Exporter 就可以完成非透明的 RPC 调用**，然后在 Invoker 的主过程上进行 Filter 拦截。
- 图中的 Consumer 和 Provider 是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓普节点，保持统一概念。
- 而 **Cluster 是外围概念**， Cluster 的目的是将多个 Invoker 伪装成一个 Invoker，这样其它人只要关注 Protocol 层 Invoker 即可，**加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，因为只有一个提供者时，是不需要 Cluster 的**。
- **Proxy 层封装了所有接口的透明化代理**，而在其它层都以 Invoker 为中心，**只有到了暴露给用户使用时，才用 Proxy 将 Invoker 转成接口，或将接口实现转成 Invoker，也就是去掉 Proxy 层 RPC 是可以 Run 的，只是不那么透明，不那么看起来像调本地服务一样调远程服务**。
- **Remoting 实现是 Dubbo 协议的实现，如果选择 RMI 协议，整个 Remoting 都不会用上**，**Remoting 内部再划为 Transport 传输层和 Exchange 信息交换层**，**Transport 层只负责单向消息传输，是对 Mina, Netty, Grizzly 的抽象**，它也可以扩展 UDP 传输，而 **Exchange 层是在传输层之上封装了 Request-Response 语义**。
- **Registry 和 Monitor 实际上不算一层**，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。



另外还有一个调用链展开的图，从下至上是消费方到服务方的一次请求的调用栈，相对简洁一些。

![img](/github/northernw.github.io/image/dubbo-extension.jpg)

再压缩一些，看这张：

![img](/github/northernw.github.io/image/send-request-process.jpg)

对应到前文提到的RPC概念、Dubbo核心功能：

带数字的1~3是Dubbo核心功能

不带数字的寻址、通信、序列化是RPC的功能

![image-20200902201050263](/github/northernw.github.io/image/image-20200902200608314.png)



### 服务导出与引用流程

Dubbo中常见的两个概念先提一下

1. SPI：Service Provider Interface服务提供接口，是JDK内置的一种服务提供发现机制，Dubbo进行了扩展。简单理解，可以动态替换接口的实现。比如有一个负载均衡的接口，有随机、轮询、最少活跃调用数等实现，在使用的时候根据传入的实现的名称来获取想要的实现。

   `LoadBalance lb = ExtensionLoader.getExtensionLoader(LoadBalance.class).getExtension("roundrobin");`

2. URL：Dubbo定义的URL类（不是Java包的URL），可以理解是一个参数类，存了服务相关的所有元信息，比如接口名称、方法有哪些、使用的协议、端口、IP等等，很多组件之间用URL进行参数的传递

   

【以下是官网提供的源码导读，基于spring的方式。代码比较早期，但核心逻辑是一致的。】

#### 服务导出

整个逻辑大致可分为三个部分

1. 第一部分是前置工作，主要用于检查参数，组装 URL
2. 第二部分是导出服务，包含导出服务到本地 (JVM)，和导出服务到远程两个过程
3. 第三部分是向注册中心注册服务，用于服务发现。

时序图

![/dev-guide/images/dubbo-export.jpg](/github/northernw.github.io/image/dubbo-export.jpg)



可以跟着时序图，关注核心的方法。



服务导出的入口是ServiceConfig的export方法。

一个ServiceConfig表示一个服务。

doExportUrls：加载注册中心（可以有多个），协议也可以有多个，在每个协议下导出服务，等价于在每个注册中心、协议下导出服务。

doExportUrlsFor1Protocol：

1. 组装要发布服务的URL，主要包括协议名称、服务所在host和port，还有服务的一些信息，如版本、时间戳、提供的方法等。

2. 导出服务

   1. 如果不配置为只导出远端，那么会先导出到本地。

      ```java
          private void exportLocal(URL url) {
              URL local = URLBuilder.from(url)
                // 这里配置协议为injvm
                      .setProtocol(LOCAL_PROTOCOL)
                      .setHost(LOCALHOST_VALUE)
                      .setPort(0)
                      .build();
            // 封装Invoker，导出到本地——本质上是根据协议injvm来区分的
            // 导向InjvmProtocol的export方法
              Exporter<?> exporter = PROTOCOL.export(
                      PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, local));
              exporters.add(exporter);
              logger.info("Export dubbo service " + interfaceClass.getName() + " to local registry url : " + local);
          }
      ```

      

   2. 然后是导出到远端。导出远端包含两个过程，服务导出和服务注册。

      ```java
      // 先为服务提供类ref封装为Invoker
      Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
      // 持有Invoker和当前ServiceConfig
      DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
      // 导出到远端，导向RegistryProtocol的export方法
      Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
      exporters.add(exporter);
      ```

      转向RegistryProtocol的export，服务导出的本质上是通过DubboProtocol的export启动了一个Netty服务（如果有多个服务导出只会启动一个服务），服务注册是向注册中心写入服务信息（例子是用zookeeper）。

      zk里注册的服务信息示例：dubbo://172.17.48.52:20880/com.alibaba.dubbo.demo.DemoService?anyhost=true&application=demo-provider&dubbo=2.0.2&generic=false&interface=com.alibaba.dubbo.demo.DemoService&methods=sayHello

   3. 导出的过程创建了一个Invoker，服务提供方创建的Invoker是对ref的代理，内部有一个字节码生成的wrapper（利用字节码生成减少反射调用的开销）

      ```java
      /** Wrapper0 是在运行时生成的，通过 Arthas 反编译得到的 */
      public class Wrapper0 extends Wrapper implements ClassGenerator.DC {
          public static String[] pns;
          public static Map pts;
          public static String[] mns;
          public static String[] dmns;
          public static Class[] mts0;
      
          // 省略其他方法
      
          public Object invokeMethod(Object object, String string, Class[] arrclass, Object[] arrobject) throws InvocationTargetException {
              DemoService demoService;
              try {
                  // 类型转换
                  demoService = (DemoService)object;
              }
              catch (Throwable throwable) {
                  throw new IllegalArgumentException(throwable);
              }
              try {
                  // 根据方法名调用指定的方法
                  if ("sayHello".equals(string) && arrclass.length == 1) {
                      return demoService.sayHello((String)arrobject[0]);
                  }
              }
              catch (Throwable throwable) {
                  throw new InvocationTargetException(throwable);
              }
              throw new NoSuchMethodException(new StringBuffer().append("Not found method \"").append(string).append("\" in class com.alibaba.dubbo.demo.DemoService.").toString());
          }
      }
      ```

      

      



#### 服务引用

消费方引用服务，根据配置有三种情形，不论哪种形式，都会得到一个Invoker实例。

1. 第一种是引用本地 (JVM) 服务
2. 第二是通过直连方式引用远程服务
3. 第三是通过注册中心引用远程服务

如果有多个服务提供者，会得到多个Invoker，这时候需要通过集群管理类Cluster将多个Invoker封装成一个。

Invoker实例具备调用本地或远程服务的能力了，但还不能直接暴露给用户使用，会对业务代码造成侵入，所以通过代理工厂类ProxyFactory为接口生成代理类，让代理类去调用Invoker逻辑，避免了对业务代码的侵入，也让框架更易用。

时序图

![/dev-guide/images/dubbo-refer.jpg](/github/northernw.github.io/image/dubbo-refer.jpg)



入口是ReferenceConfig的get方法，快速转到init方法。

主要有2个过程，处理配置和引用服务。

1. 处理配置和服务导出类似，包括配置信息检查、组装URL等。

2. 引用服务，<u>如果配置了引用本地或者在本地能找到服务，就引用本地的。否则，引用远端</u>，转向RegistryProtocol的refer方法。做这么些事情：

   1. 从注册中心获取服务提供者列表，存入RegistryDirectory

   2. 向zk注册服务消费方的信息，订阅服务提供方信息（这时候会通过DubboProtocol和提供方建立连接，如果已有别的服务引用和服务方建立了连接，可以复用这个连接）

   3. 构造路由链 routerChain，封装容错集群+负载均衡 MockClusterInvoker

   4. 封装得到Invoker（在cluster.join(directory)中）

   5. 为Invoker创建代理（PROXY_FACTORY.getProxy(invoker, ProtocolUtils.isGeneric(generic))）

      Proxy是Javassit生成的，反编译的结果：

      ```java
      public class proxy0 implements ClassGenerator.DC, EchoService, DemoService {
          // 方法数组
          public static Method[] methods;
          private InvocationHandler handler;
      
          public proxy0(InvocationHandler invocationHandler) {
              this.handler = invocationHandler;
          }
      
          public proxy0() {
          }
      
          public String sayHello(String string) {
              // 将参数存储到 Object 数组中
              Object[] arrobject = new Object[]{string};
              // 调用 InvocationHandler 实现类的 invoke 方法得到调用结果
              Object object = this.handler.invoke(this, methods[0], arrobject);
              // 返回调用结果
              return (String)object;
          }
      
          /** 回声测试方法 */
          public Object $echo(Object object) {
              Object[] arrobject = new Object[]{object};
              Object object2 = this.handler.invoke(this, methods[1], arrobject);
              return object2;
          }
      }
      ```

      



#### 调用过程

Dubbo 服务调用过程比较复杂，包含众多步骤，比如<u>发送请求、编解码</u>、服务降级、过滤器链处理、序列化、<u>线程派发以及响应</u>请求等步骤。

【会重点分析请求的发送与接收、编解码、线程派发以及响应的发送与接收等过程，至于服务降级、过滤器链和序列化可以将其当成一个黑盒，暂时忽略也没关系。】

这张图是远程调用的请求的发送和接收过程，响应的发送和接收是类似的。

![img](/github/northernw.github.io/image/send-request-process.jpg)

1. 服务消费者通过代理对象Proxy发起远程调用
2. 通过客户端Client将编码后的请求发送到服务提供方的Server
3. Server收到请求后，将数据包进行解码
4. 将解码后的请求发送到分发器Dispatcher，由分发器指派到业务线程池上
5. 线程池调用具体的服务，得到请求结果
6. （不在图上）然后反向由Server编码结果发送到Client，Client解码结果交给Proxy



##### 服务消费方

从服务引用得到的Proxy发起调用

1. 转向MockClusterInvoker（封装了降级逻辑）

2. 再到FailoverClusterInvoker（封装失败转移逻辑，简单地说，循环重试3次，每次通过负载均衡从RegistryDirectory里取一个服务提供者，发起调用，如果有调用成功的，就退出循环）

3. 2里的发起调用，转到DubboInvoker的doInvoke，会区分同步或异步调用（区别在于由谁调用 ResponseFuture 的 get 方法：同步由框架调用；异步由用户调用，通过将ResponseFuture赋值到RpcContext中供用户调用），最终请求的发送都经过ExchangeClient的send方法

4. ExchangeClient是纯网络传输（可以简单理解为Netty）的上层，封装了Request请求和Response响应语义（区分语义设置不同的消息头，编码和解码是在ExchangeCodec实现的，被Netty的decoder和encoder ChannelHandler调用），这里构造一个Request对象，放入请求数据（接口、方法、参数）

   ![image-20200904112428235](/github/northernw.github.io/image/image-20200904112428235.png)

5. 接下来再经过一些调用，就进入Netty的框架了，真正编码和发送数据（请求发送的流程就到这）

6. 看下数据包的格式。Dubbo 数据包分为消息头和消息体，消息头用于存储一些元信息，比如魔数（Magic），数据包类型（Request/Response），序列化器编号，消息体长度（Data Length）等。消息体中用于存储具体的调用消息，比如方法名称，参数列表等。

   ![img](/github/northernw.github.io/image/data-format.jpg)

   消息头的内容：

   | 偏移量(Bit) | 字段         | 取值                                                         |
   | ----------- | ------------ | ------------------------------------------------------------ |
   | 0 ~ 7       | 魔数高位     | 0xda00                                                       |
   | 8 ~ 15      | 魔数低位     | 0xbb                                                         |
   | 16          | 数据包类型   | 0 - Response, 1 - Request                                    |
   | 17          | 调用方式     | 仅在第16位被设为1的情况下有效，0 - 单向调用，1 - 双向调用    |
   | 18          | 事件标识     | 0 - 当前数据包是请求或响应包，1 - 当前数据包是心跳包         |
   | 19 ~ 23     | 序列化器编号 | 2 - Hessian2Serialization 3 - JavaSerialization 4 - CompactedJavaSerialization 6 - FastJsonSerialization 7 - NativeJavaSerialization 8 - KryoSerialization 9 - FstSerialization |
   | 24 ~ 31     | 状态         | 20 - OK 30 - CLIENT_TIMEOUT 31 - SERVER_TIMEOUT 40 - BAD_REQUEST 50 - BAD_RESPONSE ...... |
   | 32 ~ 95     | **请求编号** | 共8字节，运行时生成                                          |
   | 96 ~ 127    | 消息体长度   | 运行时计算                                                   |

以 DemoService 为例，sayHello 方法的调用路径如下：

```shell
proxy0#sayHello(String)
  —> InvokerInvocationHandler#invoke(Object, Method, Object[])
    —> MockClusterInvoker#invoke(Invocation)
      —> AbstractClusterInvoker#invoke(Invocation)
        —> FailoverClusterInvoker#doInvoke(Invocation, List<Invoker<T>>, LoadBalance)
          —> Filter#invoke(Invoker, Invocation)  // 包含多个 Filter 调用
            —> ListenerInvokerWrapper#invoke(Invocation) 
              —> AbstractInvoker#invoke(Invocation) 
                —> DubboInvoker#doInvoke(Invocation)
                  —> ReferenceCountExchangeClient#request(Object, int)
                    —> HeaderExchangeClient#request(Object, int)
                      —> HeaderExchangeChannel#request(Object, int)
                        —> AbstractPeer#send(Object)
                          —> AbstractClient#send(Object, boolean)
                            —> NettyChannel#send(Object, boolean)
                              —> NioClientSocketChannel#write(Object)
```





##### 服务提供方

Netty 检测到有数据入站后，首先会通过解码器对数据进行解码，并将解码后的数据传递给下一个入站处理器的指定方法。

DubboCodec的decode解码出一个Request对象，其中通过DecodeableRpcInvocation的decode对消息体进行解码和反序列化，得到完整的调用信息，包括attachments、方法名、参数类型和参数列表。

【解码器将数据包解析成 Request 对象后，NettyHandler 的 messageReceived 方法紧接着会收到这个对象，并将这个对象继续向后传递。这期间该对象会被依次传递给 NettyServer、MultiMessageHandler、HeartbeatHandler 以及 AllChannelHandler。最后由 AllChannelHandler 将该对象封装到 Runnable 实现类对象中，并将 Runnable 放入线程池中执行后续的调用逻辑。整个调用栈如下：】

```
NettyHandler#messageReceived(ChannelHandlerContext, MessageEvent)
  —> AbstractPeer#received(Channel, Object)
    —> MultiMessageHandler#received(Channel, Object)
      —> HeartbeatHandler#received(Channel, Object)
        —> AllChannelHandler#received(Channel, Object)
          —> ExecutorService#execute(Runnable)    // 由线程池执行后续的调用
```

1. Netty 检测到有数据入站后，首先会通过解码器对数据进行解码，并将解码后的数据传递给下一个入站处理器的指定方法。

   DubboCodec的decode解码出一个Request对象，其中通过DecodeableRpcInvocation的decode对消息体进行解码和反序列化，得到完整的调用信息，包括attachments、方法名、参数类型和参数列表。

   AllChannelHandler是Dubbo的一个线程派发策略，表示把所有消息都派发到线程池，包括请求，响应，连接事件，断开事件等。还有其他的策略，见[官网说明](http://dubbo.apache.org/zh-cn/docs/source_code_guide/service-invoking-process.html).

2. AllChannelHandler接收到数据后，转到由线程池执行ChannelEventRunnable，异步转到HeaderExchangeHandler的received进，这里构造Response，继续向后请求，并把Response写入channel。

3. 向后请求，到达DubboProtocol的reply方法，这里根据接口名称、协议端口号、协议版本号和group信息构造ServiceKey（比如com.alibaba.dubbo.demo.DemoService:1.0.0:20880），取到对应的服务提供方exporter（有个exporter的map映射），调用exporter的Invoker（服务真实实现的代理对象）得到结果。

4. 另外说一点，Netty通信可以是异步的，那请求和响应就是异步的，怎么知道当前响应是之前哪个请求的呢？Dubbo设计了一个调用编号，请求发送编号，响应带回编号，这么一一对应的。

   【一般情况下，服务消费方会并发调用多个服务，每个用户线程发送请求后，会调用不同 DefaultFuture 对象的 get 方法进行等待。如何将每个响应对象传递给相应的 DefaultFuture 对象，且不出错？答案是通过调用编号。DefaultFuture 被创建时，会要求传入一个 Request 对象。DefaultFuture 可从 Request 对象中获取调用编号，并将 <调用编号, DefaultFuture 对象> 映射关系存入到静态 Map 中，即 FUTURES。线程池中的线程在收到 Response 对象后，会根据 Response 对象中的调用编号到 FUTURES 集合中取出相应的 DefaultFuture 对象，然后再将 Response 对象设置到 DefaultFuture 对象中。最后再唤醒用户线程，这样用户线程即可从 DefaultFuture 对象中获取调用结果了】

调用过程：

```shell
ChannelEventRunnable#run()
  —> DecodeHandler#received(Channel, Object)
    —> HeaderExchangeHandler#received(Channel, Object)
      —> HeaderExchangeHandler#handleRequest(ExchangeChannel, Request)
        —> DubboProtocol.requestHandler#reply(ExchangeChannel, Object)
          —> Filter#invoke(Invoker, Invocation)
            —> AbstractProxyInvoker#invoke(Invocation)
              —> Wrapper0#invokeMethod(Object, String, Class[], Object[])
                —> DemoServiceImpl#sayHello(String)
```

调用编号：![img](/github/northernw.github.io/image/request-id-application.jpg)



小结一下：

1. 比较重要的编码和解码在DubboCodec
2. 重要的rpc中转在DubboProtocol
   1. 消费方调用RegistryProtocol的本质是有注册中心的多个服务方，待路由和负载均衡后还是经过DubboProtocol进行调用
   2. 提供方通过RegistryProtocol，也是在注册中心的范畴上注册服务，再通过DubboProtocol发布服务（发布服务本质上是在DubboProtocol上缓存了一个接口的实现类的代理，让DubboProtocol可以根据接口信息调用到实现类）
3. 请求和响应的写数据通过Netty channel的read和send，属于Netty的知识点
4. 提供方和消费方的代理类不太一样
   1. 提供方是实现类的一个字节码生成的代理类，为了节省反射调用的开销（通过传入的方法名和实现类中的方法名做equals，显式调用实现类的该方法，而不是通过Java提供的反射调用）
   2. 消费方的代理类，说白了是对RegistryProtocol/DubboProtocol（或InjvmProtocol）的封装，调用代理类就发起远端rpc（或本地）的调用



### 参考链接和书目

1. Dubbo中文官网：http://dubbo.apache.org/zh-cn/docs/user/quick-start.html
2. 翟陆续.深度剖析Apache Dubbo核心技术内幕[M]. 北京：电子工业出版社，2019-12.



### 待续

难点

1. SPI机制，原理比较好理解，实现类是javasisst字节码生成的，可读性不是太好，但以理解原理为先。
2. 分层比较多，概念有不太统一的地方，会增加理解的难度



###### 服务目录

http://dubbo.apache.org/zh-cn/docs/source_code_guide/directory.html

服务目录中存储了服务提供者有关的信息，通过服务目录，服务消费者可获取到服务提供者的信息，比如 ip、端口、服务协议等。





###### 服务路由

http://dubbo.apache.org/zh-cn/docs/source_code_guide/router.html

比如有黑名单。

路由规则决定了服务消费者的调用目标，即规定了服务消费者可调用哪些服务提供者。Dubbo 目前提供了三种服务路由实现，分别为条件路由 ConditionRouter、脚本路由 ScriptRouter 和标签路由 TagRouter。其中条件路由是我们最常使用的，标签路由是一个新的实现，暂时还未发布，该实现预计会在 2.7.x 版本中发布。



###### 服务集群

http://dubbo.apache.org/zh-cn/docs/source_code_guide/cluster.html

集群工作过程可分为两个阶段，第一个阶段是在服务消费者初始化期间，集群 Cluster 实现类为服务消费者创建 Cluster Invoker 实例，即上图中的 merge 操作。第二个阶段是在服务消费者进行远程调用时。以 FailoverClusterInvoker 为例，该类型 Cluster Invoker 首先会调用 Directory 的 list 方法列举 Invoker 列表（可将 Invoker 简单理解为服务提供者）。Directory 的用途是保存 Invoker，可简单类比为 `List<Invoker>`。其实现类 RegistryDirectory 是一个动态服务目录，可感知注册中心配置的变化，它所持有的 Invoker 列表会随着注册中心内容的变化而变化。每次变化后，RegistryDirectory 会动态增删 Invoker，并调用 Router 的 route 方法进行路由，过滤掉不符合路由规则的 Invoker。当 FailoverClusterInvoker 拿到 Directory 返回的 Invoker 列表后，它会通过 LoadBalance 从 Invoker 列表中选择一个 Invoker。最后 FailoverClusterInvoker 会将参数传给 LoadBalance 选择出的 Invoker 实例的 invoke 方法，进行真正的远程调用。



###### 负载均衡

http://dubbo.apache.org/zh-cn/docs/source_code_guide/loadbalance.html

LoadBalance 中文意思为负载均衡，它的职责是将网络请求，或者其他形式的负载“均摊”到不同的机器上。避免集群中部分服务器压力过大，而另一些服务器比较空闲的情况。通过负载均衡，可以让每台服务器获取到适合自己处理能力的负载。在为高负载服务器分流的同时，还可以避免资源浪费，一举两得。
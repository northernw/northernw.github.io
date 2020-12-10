---
title: Netty学习笔记
tags:
  - NOTE
  - Netty
  - 学习笔记
categories:
  - NOTE
date: 2020-11-18 11:13:20
---



###### Java NIO

https://www.javadoop.com/post/java-nio

三大组件

1. Buffer

   本质是内存中的一块，将数据写入这块内存，从这块内存获取数据

   核心是ByteBuffer，也最常使用

2. Channel

   通道，数据来源或数据写入的目的地

   与Buffer打交道，读操作时将Channel的数据填充到Buffer中，写操作将Buffer的数据写入Channel。

   读操作：`channel.read(buffer)`

   写操作：`channel.write(buffer)`

3. Selector

   多路复用，一个线程管理多个Channel

   将Channel注册到Selector上（带上需要监听的事件类型s）

   四种事件：

   1. SelectionKey.OP_READ 对应 00000001，通道中有数据可以进行读取
   2. SelectionKey.OP_WRITE 对应 00000100，可以往通道中写入数据
   3. SelectionKey.OP_CONNECT 对应 00001000，成功建立 TCP 连接
   4. SelectionKey.OP_ACCEPT 对应 00010000，接受 TCP 连接



> ServerSocketChannel不和Buffer交互，不处理实际数据，一旦接收到请求后，实例化SocketChannel，之后在这个连接通道上的数据传递它就不管了，继续监听端口，等待下一个连接。

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(8080));
while(true){
  SocketChannel socketChannel = serverSocketChannel.accept();
}
```



###### Java的非阻塞IO和异步IO

https://www.javadoop.com/post/nio-and-aio

简单地说，ServerSocketChannel、Selector这种多路复用的，是非阻塞，non-blocking。

AsynchronousServerSocketChannel是异步的，有Future和回调函数两种方式。





##### Netty



###### Channel



###### Future和Promise



###### Pipeline和Inbound、Outbound



###### EventLoopGroup和EventLoop

Netty线程池指NioEventLoopGroup，线程池中的单个线程是NioEventLoop。
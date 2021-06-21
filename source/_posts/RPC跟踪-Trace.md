---
title: RPC跟踪-Trace
tags:
  - null
categories:
  - null
date: 2021-06-11 16:41:15
---



简单记录一下原理

1. web端的interceptor配置TraceInterceptor，在preHandler里开启一个TraceMessage，配置了两个tracer（log + rpc），分别写入traceId信息
   1. log: MDC.put("traceId", traceId);
   2. rpc: setSessionAttribute("traceId", traceId);
2. 因此呢
   1. 日志的patten中配置了`[%X{traceId}]`会取到MDC中的traceId
   2. rpc发送的请求中会带上traceId，在rpc接收端配置一个filter，开启一个TraceMessage，同1里的作用
3. traceId可以使用uuid `UUID.randomUUID()`

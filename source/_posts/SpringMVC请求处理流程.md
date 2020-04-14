---
title: SpringMVC请求处理流程
tags:
  - SpringMVC
  - web
categories:
  - null
date: 2020-04-14 16:08:43
---



小结

1. getHandler取到一个HandlerExecutionChain mappedHandler，包含URL对应的controller方法HandlerMethod，和一些interceptors
2. HandlerMethod取到对应的handlerAdapter，数据绑定就再这个ha中做的
3. mappedHandler执行拦截器的preHandle
4. handlerAdapter执行controller方法，包含请求前的数据绑定（数据转换），和请求后的数据转换
5. mappedHandler执行拦截器的postHandle
6. 以上过程如果有抛出异常，由全局异常处理器来处理
7. mappedHandler触发拦截器的afterCompletion





getHandler

HandlerMapping是用来找Handler的

第一个是RequestMappingHandlerMapping，找到的Handler是HandlerMethod，也就是URL对应的controller的方法

寻找interceptorList（一系列拦截器，包括自定义的业务拦截器，系统预定义的一些拦截器？），与HandlerMethod一起分装在HandlerExecutionChain里

![image-20200414160859525](/github/northernw.github.io/image/image-20200414160859525.png)

![image-20200414161600585](/github/northernw.github.io/image/image-20200414161600585.png)

![image-20200414161303329](/github/northernw.github.io/image/image-20200414161303329.png)





根据Handler获取HandlerAdapte

![image-20200414162424370](/github/northernw.github.io/image/image-20200414162424370.png)

![image-20200414162348657](/github/northernw.github.io/image/image-20200414162348657.png)
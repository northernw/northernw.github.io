---
title: spring-mvc笔记
tags:
  - null
categories:
  - null
date: 2019-10-25 16:55:41
---



## request处理与找到method

`getHandler`返回需要的method，在这里就是`UserController`的`query`方法。

![image-20191025173756807](../../image/image-20191025173756807.png)





入参和出参都是在进入method invoke后才处理的

![image-20191025180008141](../../image/image-20191025180008141.png)



真正执行数据绑定的地方

![image-20191025180826737](../../image/image-20191025180826737.png)







## invoke与response处理

在反射执行完controller中的方法后，通过`selectHandler`找到匹配的处理返回结果的Processor。例如返回结果是json或xml格式的，都是在这个processor中。和`@ResponseBody`的数据绑定是同一个。

![image-20191025165729754](../../image/image-20191025165729754.png)





![img](https://upload-images.jianshu.io/upload_images/5220087-3c0f59d3c39a12dd.png)


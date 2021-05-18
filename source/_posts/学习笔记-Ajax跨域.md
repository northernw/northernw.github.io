---
title: 学习笔记-Ajax跨域
tags:
  - Ajax跨域
categories:
  - 学习笔记
date: 2021-04-28 15:30:30
typora-copy-images-to: ../../image
---



### 产生跨域的原因？

同时满足以下条件：

1. 浏览器限制
2. 跨域：发出去的请求不是本域的，域名+端口
3. XHR（XMLHttpRequest）请求：发出去的是XHR请求



### 解决思路

1. 浏览器 -> 禁用跨域检查
2. XHR -> 换JSONP（json for pending）请求，效果不好，不满足现在的开发要求

3. 解决跨域

“A域名调用B域名时，在B域名返回的信息里加入一些字段，告诉浏览器允许A域名调用。”

被调用方修改：支持跨域

调用方修改：隐藏跨域，A域名发出的请求，通过代理转成B域名的请求，让浏览器认为是同一个域名的请求

<img src="/github/northernw.github.io/image/image-20210428154025390.png" alt="image-20210428154025390" style="zoom:33%;" />





### 简单请求和非简单请求

简单请求直接发送

非简单请求会先发送预检命令，预检通过后再发送非简单请求

<img src="/github/northernw.github.io/image/image-20210428160650040.png" alt="image-20210428160650040" style="zoom:50%;" />



### 被调用方允许跨域

#### 应用服务器允许跨域

1. 简单请求只需要origin和method两个配置
2. 非简单请求要加headers
3. 带cookie请求的origin要完全匹配，并且加上credentials

<img src="/github/northernw.github.io/image/image-20210428161948808.png" alt="image-20210428161948808" style="zoom:50%;" />



#### Nginx允许跨域

<img src="/github/northernw.github.io/image/image-20210428164638518.png" alt="image-20210428164638518" style="zoom:50%;" />



#### Spring框架允许跨域

使用`@CrossOrigin`注解

![image-20210428165218140](/github/northernw.github.io/image/image-20210428165218140.png)

### 虚拟主机

多个域名指向同一个服务器，服务器根据域名把请求转到不同的应用服务器，看起来有多个主机，其实只有一个。
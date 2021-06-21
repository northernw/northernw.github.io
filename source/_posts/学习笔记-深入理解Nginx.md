---
title: 学习笔记-深入理解Nginx
tags:
  - Nginx
categories:
  - 学习笔记
date: 2021-06-21 16:28:46
---





### Nginx是什么？

轻量级、高性能的web服务器。

俄罗斯，Igor Sysoev（伊戈尔·西索夫），2004年。

### 为什么选择Nginx？

有以下特点：

1. 更快：正常情况下，单次请求更快；高并发时，相比其他web服务器更快响应。
2. 高扩展性：由不同功能、不同层次、不同类型且耦合度极低的模块组成。
3. 高可靠性：核心框架设计优秀，模块设计简单；官方提供的模块很稳定，每个worker进程相对独立，可快速failover。
4. 低内存消耗：1万个非活跃的HTTP keep alive连接在Nginx中仅消耗2.5M内存
5. 单机支持10万以上的并发连接
6. 热部署：不间断服务的情况下升级可执行文件、更新配置文件、更换日志文件
7. 最自由的BSD许可协议：免费使用，可直接使用或修改源码

核心理由：支持高并发请求的同时，保持高效的服务。高并发、高可靠性。

### 常用命令

1. 启动 `nginx`

2. 另行指定配置文件 `nginx -c /tmp/ngxin.conf`

3. 检测配置文件是否正确 `nginx -t`

4. 显示版本信息 `nginx -v`

5. 快速停止/强制停止  `nginx -s stop`

6. 优雅停止 `nginx -s quit`

7. 重读配置文件并生效 `nginx -s reload`

8. 平滑升级

   ```
   kill -s SIGUSR2 <nginx master pid>
   nginx
   kill -s SIGQUIT <nginx old master pid>
   ```

   

## Nginx的配置

### Nginx进程间的关系

一个master进程管理多个worker进程，一般情况下，worker进程数量与服务器上的CPU核数相同。

一个worker进程可以同时处理多个请求，只受限于内存大小。

worker数与核数相同，每个worker绑定一个核心，进程间切换的代价最小。

worker进程不会进入睡眠状态。

Apache服务器，一个进程只能处理一个请求，需要开启成百个进程来支持高并发，进程切换代价大。

### Nginx配置的通用语法

#### 块配置项

块配置项+一对花括号，注释用`#`，支持多种单位

1. events
2. http
3. server
4. location
5. upstream

空间单位：K或者k 千字节，M或者m 兆字节

时间单位：ms/s/m/h/d/w（7天）/M（30天）/y（365天）

每行配置的结尾要加分号。

#### 使用变量

有些模块支持使用变量，变量前加`$`，比如日志记录部分：

```
log_format main '$remote_addr - $remote-user [$time_local] "$request" $status $byte_sent "$$http_referer" "$http_user_agent" "$http_x_forwarded_for"';
```



### 虚拟主机与请求分发

多个主机域名对应同一个IP地址。

nginx.conf中按照server_name（对应请求中的域名）和server块来定义虚拟主机，每个server块就是一个虚拟主机，只处理与之相应的主机域名请求。

（1）监听端口

```
语法：listen address:port [default(deprecated)|default_server|[backlog=num|rcvbuf=size|sndbbuf=size|accept_filter=filter|bind|ipv6only=[on|off]|ssl]]
默认：listen 80;
```

default|default_server：所在server作为整个web服务默认的server块。兜底server。

listen后面可以只加IP地址、端口或主机名。

不加端口时，默认监听80端口（不确定最新版本是80还是8080）。

（2）主机名称

```
语法：server_name name [...];
默认：server_name "";
```

server_name与Host的匹配优先级：

1. 字符串完全匹配的server_name，如`www.testweb.com`
2. 通配符在前的server_name，如`*.testweb.com`
3. 通配符在后的server_name，如`www.testweb.*`
4. 正则表达式匹配的server_name，如`~^\.testweb\.com$`

如果Host与所有server_name都不匹配，则：

1. 选择在listen配置后加入default|default_server的server块
2. 找到匹配listen端口的第一个server块

（3）location

```
语法：location [=|~|~*|^~|@] /uri {...}
```

匹配规则：

1. =：把URI作为字符串，需要与参数中的uri完全匹配
2. ~：大小写敏感
3. ~*：忽略大小写
4. ~^：只需要前半部分匹配uri（左匹配）
5. @：仅用于Nginx服务内部请求之间的重定向（不直接处理用户请求）

location是有序的，当请求能匹配多个location时，请求会被第一个location处理。

只能表达“如果匹配...则...”，不能表达“如果不匹配...则...”。

`location / {...}`可以匹配所有请求，可以作为托底处理。





------

深入理解Nginx

模块开发与架构解析，第二版

陶辉

2017.机械工业出版社

[1]陶辉. 深入理解Nginx.第2版[M]. 机械工业出版社, 2016.




---
title: 读书笔记【持续演进的Cloud Native：云原生架构下微服务最佳实践】
tags:
  - null
categories:
  - null
date: 2019-11-25 11:58:04
---










# 摘录

## DevOps简介
原文地址：https://www.cnblogs.com/liufei1983/p/7152013.html

实现DevOps需要什么？

**硬性要求：工具上的准备**

上文提到了工具链的打通，那么工具自然就需要做好准备。现将工具类型及对应的不完全列举整理如下：

- 代码管理（SCM）：**GitHub**、GitLab、BitBucket、SubVersion
- 构建工具：**Ant**、Gradle、**maven**
- 自动部署：Capistrano、CodeDeploy
- 持续集成（CI）：Bamboo、Hudson、Jenkins
- 配置管理：Ansible、Chef、Puppet、SaltStack、ScriptRock GuardRail
- 容器：**Docker**、LXC、第三方厂商如AWS
- 编排：Kubernetes、Core、Apache Mesos、DC/OS
- 服务注册与发现：**Zookeeper**、etcd、Consul
- 脚本语言：python、ruby、shell
- 日志管理：ELK、Logentries
- 系统监控：Datadog、Graphite、Icinga、Nagios
- 性能监控：AppDynamics、New Relic、Splunk
- 压力测试：JMeter、Blaze Meter、loader.io
- 预警：PagerDuty、pingdom、厂商自带如AWS SNS
- HTTP加速器：Varnish
- 消息总线：ActiveMQ、SQS
- 应用服务器：Tomcat、JBoss
- Web服务器：Apache、Nginx、IIS
- 数据库：MySQL、Oracle、PostgreSQL等关系型数据库；cassandra、mongoDB、redis等NoSQL数据库
- 项目管理（PM）：Jira、Asana、Taiga、Trello、Basecamp、Pivotal Tracker

在工具的选择上，需要结合公司业务需求和技术团队情况而定。（注：更多关于工具的详细介绍可以参见此文：51 Best DevOps Tools for #DevOps Engineers）







# 概念

CNCF（Cloud Native Computing Foundation）云原生计算基金会

Istio 2017.05.24 Google&IBM 第二代Service Mesh

conduit 201712.05 Buoyant 第二代Service Mesh



DevOps is a set of practices that combines software development and information-technology operations which aims to shorten the systems development life cycle and provide continuous delivery with high software quality.

DevOps 是一个完整的面向IT运维的工作流，以 IT 自动化以及持续集成（CI）、持续部署（CD）为基础，来优化程式开发、测试、系统运维等所有环节。



Service Mesh 服务网格，服务间通信的基础设施层。可以将它比作是**应用程序或者说微服务间的 TCP/IP，负责服务之间的网络调用、限流、熔断和监控**。



Serviceless

Serverless不代表再也不需要服务器了，而是说：开发者再也不用过多考虑服务器的问题，计算资源作为服务而不是服务器的概念出现。

Serverless是一种构建和管理基于微服务架构的完整流程，允许在服务部署级别而不是服务器部署级别来管理应用部署，甚至可以管理某个具体功能或端口的部署，这就能让开发者快速迭代，更快速地开发软件。



Sofa

SOFAStack™（Scalable Open Financial Architecture Stack）是一套用于快速构建**金融级分布式架构的中间件**，也是在金融场景里锤炼出来的最佳实践。
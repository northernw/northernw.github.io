---
title: UML-学习笔记
tags:
  - null
categories:
  - null
date: 2021-02-25 16:19:47
typora-copy-images-to: ../../image
---

<img src="/github/northernw.github.io/image/02-uml-diagram-types.png" alt="UML 图表的种类"  />



## 结构型

### 类图 Class Diagram

对象的类型，以及它们之间存在的各种静态关系。

![类图](/github/northernw.github.io/image/03-class-diagram-example.png)



### 组件图 Component Diagram

描绘组件如何连接在一起，以形成更大的组件或者软件系统。

https://www.visual-paradigm.com/guide/uml-unified-modeling-language/what-is-component-diagram/

![组件图](/github/northernw.github.io/image/04-component-diagram-example.png)

![Component Diagram at a glance](/github/northernw.github.io/image/02-component-diagram-overview.png)

### 部署图 Deployment Diagram

表达了硬件配置以及和软件组件之间的关系。

![部署图](/github/northernw.github.io/image/05-deployment-diagarm.png)

### 对象图 Object Diagram

![对象图示例](/github/northernw.github.io/image/07-object-diagram-example.png)

### 包图 Package Diagram

包之间的依赖关系

![包图](/github/northernw.github.io/image/08-package-diagram.png)

### 组合结构图 Composite Structure Diagram

不懂..

类的内部结构，和这个结构所实现的协作

UML2.0

![复合结构图](/github/northernw.github.io/image/09-composite-structure-diagram.png)

### 轮廓图 Profile Diagram

不懂+2.

创建特定于域和平台的原型，并定义它们之间的关系

![轮廓图](/github/northernw.github.io/image/10-profile-diagram.png)



## 行为型

### 用例图 Use Case Diagram

![用例图](/github/northernw.github.io/image/11-use-case-diagram.png)

### 活动图 Activity Diagram

活动图用于展示工作流程，它支持选择 (Choice)，迭代 (Iteration)和并发 (Concurrency)

![活动图](/github/northernw.github.io/image/12-activity-diagram.png)

### 状态机图 State Machine Diagram

状态图描绘允许的状态和转换以及影响这些转换的事件，它有助于可视化对象的整个生命周期，从而更好地理解以状态主導 (State-based) 的系统。

![状态机图](/github/northernw.github.io/image/13-state-machine-diagram.png)

### 序列图 Sequence Diagram

序列图根据时间序列展示对象如何进行协作

![序列图](/github/northernw.github.io/image/14-sequence-diagram.png)

### 通讯图 Communication Diagram

与序列图类似，通訊圖也用于模拟用例的动态行为。与序列图相比，通訊圖更侧重于显示对象的协作而不是时间顺序。它们实际上在语义上是等价的。

![通訊圖](/github/northernw.github.io/image/15-activity-diagram.png)

### 交互概述图 Interaction Overview Diagram

交互概述图侧重于交互控制流程的概述，它是活动图的变体。

![交互概览图](/github/northernw.github.io/image/16-interaction-overview-diagram.png)

### 时序图 Timing Diagram

看不懂..

时序图显示了既定时间内对象的行为。时序图是序列图的一种特殊形式，它俩之间的差异是轴反转，时间从左到右增加，生命线显示在垂直排列的独立隔间中。

![时序图> 						<div class=](/github/northernw.github.io/image/17-timing-diagram.png)
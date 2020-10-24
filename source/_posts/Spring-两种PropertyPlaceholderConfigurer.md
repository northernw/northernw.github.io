---
title: Spring-两种PropertyPlaceholderConfigurer
tags:
  - PropertyPlaceholderConfigurer
categories:
  - Spring
date: 2020-05-19 11:26:24
---



![image-20200519122103524](../../image/image-20200519122103524.png)

![image-20200519122032838](../../image/image-20200519122032838.png)





加载配置文件的地方

![image-20200519123638203](../../image/image-20200519123638203.png)

![image-20200519123710774](../../image/image-20200519123710774.png)





# Case Study

项目中注册了两个PropertyPlaceholderConfigurer，一个是xml中配置的`PropertyPlaceholderConfigurer`，显示设置`ignoreUnresolvablePlaceholders`为true（忽略找不到的占位符）；另一个是第三方组件注册的`PropertySourcesPlaceholderConfigurer`，默认配置。

```xml
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="ignoreResourceNotFound" value="true"/>
        <property name="properties" ref="prop"/>
        <property name="locations">
            <list>
                <value>classpath:config/*.properties</value>
            </list>
        </property>
    </bean>
```



![image-20200519113348338](../../image/image-20200519113348338.png)
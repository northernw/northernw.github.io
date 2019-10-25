---
title: spring design pattern
tags:
  - null
categories:
  - null
date: 2019-10-23 19:03:20
---

# 设计模式

## 简单工厂模式（非GoF）

简单工厂是指，只用一个具体工厂就可以完成所有产品的生产。

比如`BeanFactory`生产出各种类型的bean。

通俗的说，就是创建产品的接口里，`if else`判断要哪种类型的产品，就创建哪种类型的产品。

`BeanFactory`是用一个map维护了产品和产品的类型。

ps：简单工厂不属于GoF，原因是会破坏开闭原则，说来就是增加产品时需要增加`if`判断。但用map和自动的类型注册，就不会破坏啦。也是很好用的模式。





##工厂方法模式

`FactoryBean `

spring中，当bean的初始化比较复杂，有大量工作要做时使用。

一个抽象工厂有多个具体工厂，对应着抽象产品的多个具体产品创建。

缺点，也是特点，每多一种产品，就要加一个具体工厂。



工厂方法和简单工厂如何区分呢？

删掉抽象工厂，只用具体工厂就能生产所有产品，这时就可退化成简单工厂。

工厂方法一般是有一个抽象工厂，多个具体工厂的。





## 抽象工厂模式

抽象工厂是工厂方法的升级版，工厂提供多类产品的创建，就是说，抽象产品有多个。

新增一个具体工厂时，不修改原代码，满足开闭原则。

要新增一类产品时，就要在每个工厂里增加创建这类产品的接口，不满足开闭原则。



[抽象工厂模式（详解版）](http://c.biancheng.net/view/1351.html)

> 抽象工厂模式最早的应用是用于创建属于不同操作系统的视窗构件。如 java 的 AWT 中的 Button 和 Text 等构件在 Windows 和 UNIX 中的本地实现是不同的。



抽象工厂和工厂方法又怎么区分呢？

工厂方法提供一类/同类产品的生产，抽象工厂提供多类产品。





Ps:

[Spring框架中的设计模式（一）](http://blog.didispace.com/spring-design-partern/)这篇文章提到`BeanFactory`是抽象工厂，我理解不太对。`BeanFactory`是工厂方法，因为`BeanFactory`只提供了一类产品。

> 在`BeanFactory`的实现中，我们可以区分：`ClassPathXmlApplicationContext`，`XmlWebApplicationContext`，`StaticWebApplicationContext`，`StaticPortletApplicationContext`，`GenericApplicationContext`，`StaticApplicationContext`。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"file:test-context.xml"})
public class TestProduct {
         
  @Autowired
  private BeanFactory factory;
   
  @Test
  public void test() {
    System.out.println("Concrete factory is: "+factory.getClass());
    assertTrue("Factory can't be null", factory != null);
    ShoppingCart cart = (ShoppingCart) factory.getBean("shoppingCart");
    assertTrue("Shopping cart object can't be null", cart != null);
    System.out.println("Found shopping cart bean:"+cart.getClass());
  }
}
```

> 在这种情况下，抽象工厂由BeanFactory接口表示。具体工厂是在第一个System.out中打印的，是**org.springframework.beans.factory.support.DefaultListableBeanFactory**的实例。它的抽象产物是一个对象。在我们的例子中，具体的产品就是被强转为ShoppingCart实例的抽象产品（Object）。

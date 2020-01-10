本文由群里的绪扬同学投稿，在此，也欢迎更多的同学来投稿。

**# 1、译者的话**

之前去京东面试，被问到 AOP 相关的问题，之前一直没有系统地学习相关的知识，答得不是很好。趁着假期，找了一下相关的资料，CSDN上有很多不错的文章，看了之后对 AOP 有比较好的理解了。然后 Google 了一下 AOP 相关面试题「AOP interview」，搜出来的第一条结果是一个叫 How To Do In Java 的网站上的一篇文章 Top Spring AOP Interview Questions with Answers。

![img](https://pic2.zhimg.com/80/v2-962732dea3429d2cd979842ba47cb4f5_hd.jpg)

看了一下，实话说，写得并不是很简单易懂，只是介绍性的文章而已。但是通篇看下来，基本的知识点也都覆盖到了。文章不是很长，所以为了加深印象，顺便把它翻译出来，以供大家参考。

**# 2、Spring AOP 最热门面试题及答案**

看完 Spring 核心面试题之后，让我们来看一下 Spring AOP 面试题，这个你可能会在下一次技术面试的时候遇到。同样的，请尽管提出一些这个帖子相关的新的问题来，我会把它们包含进来，以让更多的读者受益。

**# 3、 内容大纲**

- 描述一下Spring AOP？
- 在Spring AOP中关注点(concern)和横切关注点(cross-cutting concern)有什么不同？
- AOP有哪些可用的实现？
- Spring中有哪些不同的通知类型(advice types)？
- Spring AOP 代理是什么？
- 引介(Introduction)是什么？
- 连接点(Joint Point)和切入点(Point Cut)是什么？
- 织入（Weaving）是什么？

**# 4、描述一下Spring AOP**

Spring AOP(Aspect Oriented Programming，面向切面编程)是OOPs(面向对象编程)的补充，它也提供了模块化。在面向对象编程中，关键的单元是对象，AOP的关键单元是切面，或者说关注点（可以简单地理解为你程序中的独立模块）。一些切面可能有集中的代码，但是有些可能被分散或者混杂在一起，例如日志或者事务。这些分散的切面被称为横切关注点。一个横切关注点是一个可以影响到整个应用的关注点，而且应该被尽量地集中到代码的一个地方，例如事务管理、权限、日志、安全等。

AOP让你可以使用简单可插拔的配置，在实际逻辑执行之前、之后或周围动态添加横切关注点。这让代码在当下和将来都变得易于维护。如果你是使用XML来使用切面的话，要添加或删除关注点，你不用重新编译完整的源代码，而仅仅需要修改配置文件就可以了。

Spring AOP通过以下两种方式来使用。但是最广泛使用的方式是Spring AspectJ 注解风格(Spring AspectJ Annotation Style)。

- 使用AspectJ 注解风格
- 使用Spring XML 配置风格

**# 5、在Spring AOP中关注点和横切关注点有什么不同？**

关注点是我们想在应用的模块中实现的行为。关注点可以被定义为：我们想实现以解决特定业务问题的方法。比如，在所有电子商务应用中，不同的关注点（或者模块）可能是库存管理、航运管理、用户管理等。

横切关注点是贯穿整个应用程序的关注点。像日志、安全和数据转换，它们在应用的每一个模块都是必须的，所以他们是一种横切关注点。

**# 6、AOP有哪些可用的实现？**

基于Java的主要AOP实现有，如下。

- AspectJ
- Spring AOP
- JBoss AOP

在维基百科上你可以找到一个AOP实现的大列表。

![img](https://pic2.zhimg.com/80/v2-d822b60b8c34617b6824814c52f0e02d_hd.jpg)

**# 7、Spring中有哪些不同的通知类型**

通知(advice)是你在你的程序中想要应用在其他模块中的横切关注点的实现。Advice主要有以下5种类型。

前置通知(Before Advice): 在连接点之前执行的Advice，不过除非它抛出异常，否则没有能力中断执行流。使用 @Before注解使用这个Advice。

返回之后通知(After Retuning Advice): 在连接点正常结束之后执行的Advice。例如，如果一个方法没有抛出异常正常返回。通过 @AfterReturning`关注使用它。

抛出（异常）后执行通知(After Throwing Advice): 如果一个方法通过抛出异常来退出的话，这个Advice就会被执行。通用 @AfterThrowing`注解来使用。

后置通知(After Advice): 无论连接点是通过什么方式退出的(正常返回或者抛出异常)都会执行在结束后执行这些Advice。通过@After注解使用。

围绕通知(Around Advice): 围绕连接点执行的Advice，就你一个方法调用。这是最强大的Advice。通过 @Around注解使用。

**# 8、Spring AOP 代理是什么？**

代理是使用非常广泛的设计模式。简单来说，代理是一个看其他像另一个对象的对象，但它添加了一些特殊的功能。

Spring AOP是基于代理实现的。AOP 代理是一个由 AOP 框架创建的用于在运行时实现切面协议的对象。

Spring AOP默认为 AOP 代理使用标准的 JDK 动态代理。这使得任何接口（或者接口的集合）可以被代理。Spring AOP 也可以使用 CGLIB 代理。这对代理类而不是接口是必须的。

如果业务对象没有实现任何接口那么默认使用CGLIB。

**# 9、引介(Introduction)是什么？**

引介让一个切面可以声明被通知的对象实现了任何他们没有真正实现的额外接口，而且为这些对象提供接口的实现。

使用 @DeclareParaents`注解来生成一个引介。

更多详情，请参考官方文档。

**# 10、连接点(Joint Point)和切入点(Point cut)是什么？**

连接点是程序执行的一个点。例如，一个方法的执行或者一个异常的处理。在 Spring AOP 中，一个连接点总是代表一个方法执行。举例来说，所有定义在你的 EmpoyeeManager接口中的方法都可以被认为是一个连接点，如果你在这些方法上使用横切关注点的话。

切入点是一个匹配连接点的断言或者表达式。Advice 与切入点表达式相关联，并在切入点匹配的任何连接点处运行（比如，表达式 execution(* EmployeeManager.getEmployeeById(...)) 可以匹配 EmployeeManager接口的getEmployeeById()）。由切入点表达式匹配的连接点的概念是 AOP 的核心。Spring 默认使用 AspectJ 切入点表达式语言。

**# 11、什么是织入(weaving)？**

Spring AOP 框架仅支持有限的几个 AspectJ 切入点的类型，它允许将切面运用到在 IoC 容器中声明的 bean 上。如果你想使用额外的切入点类型或者将切面应用到在 Spring IoC 容器外部创建的类，那么你必须在你的 Spring 程序中使用 AspectJ 框架，并且使用它的织入特性。

织入是将切面与外部的应用类型或者类连接起来以创建通知对象(adviced object)的过程。这可以在编译时(比如使用 AspectJ 编译器)、加载时或者运行时完成。Spring AOP 跟其他纯 Java AOP 框架一样，只在运行时执行织入。在协议上，AspectJ 框架支持编译时和加载时织入。

AspectJ 编译时织入是通过一个叫做ajc特殊的 AspectJ 编译器完成的。它可以将切面织入到你的 Java 源码文件中，然后输出织入后的二进制 class 文件。它也可以将切面织入你的编译后的 class 文件或者 Jar 文件。这个过程叫做后编译时织入(post-compile-time weaving)。在 Spring IoC 容器中声明你的类之前，你可以为它们运行编译时和后编译时织入。Spring 完全没有被包含到织入的过程中。更多关于编译时和后编译时织入的信息，请查阅 AspectJ 文档。

AspectJ 加载时织入(load-time weaving, LTW)在目标类被类加载器加载到JVM时触发。对于一个被织入的对象，需要一个特殊的类加载器来增强目标类的字节码。AspectJ 和 Spring 都提供了加载时织入器以为类加载添加加载时织入的能力。你只需要简单的配置就可以打开这个加载时织入器。

如果你有更好的方案，欢迎在留言区一起探讨。
---
title: mockito使用
tags:
  - mockito
  - java
  - ut
categories:
  - NOTE
date: 2021-01-26 16:15:02
---

# Mockito在Spring中的依赖注入





## mockito引入

```xml
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>3.7.7</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.9.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.3.9.RELEASE</version>
            <scope>provided</scope>
        </dependency>
```







## Mockito与testng

### maven依赖

```xml
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.1.0</version>
            <scope>test</scope>
        </dependency>
```



### 使用Demo

```java
@ContextConfiguration(locations = "classpath:spring-config.xml")
public class UserServiceTestngTest extends AbstractTestNGSpringContextTests {
    @Mock // 1. 声明需要mock的对象
    private UserDao userDao;

    @Autowired // 注意：spring容器初始化时会对UserService进行依赖注入，所以依赖实现还是得存在的，在openMocks中再对依赖进行替换
    @InjectMocks // 2. 声明需要进行依赖注入的对象
    private UserService userService;


    @BeforeClass
    public void beforeClass() {
        // 3. 对2中的属性注入1中声明的mock对象
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void get() {
        // 4. 配置mock
        Mockito.when(userDao.getUser()).thenReturn(new User("lily", 16));

        User user = userService.get();
        System.out.println(user);
    }
}
```







## Mockito与junit

### maven依赖

```xml
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>
```



### 使用Demo

使用要点与testng的demo一致

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-config.xml")
public class UserServiceTest {
    @Mock
    private UserDao userDao;

    @Autowired
    @InjectMocks
    private UserService userService;

    @Before
    public void before() {
        MockitoAnnotations.openMocks(this);
    }


    @Test
    public void get() {
        Mockito.when(userDao.getUser()).thenReturn(new User("lily", 16));

        User user = userService.get();
        System.out.println(user);
    }
}
```



## Mockito遇上Spring Aop失效？

当`UserService`有aop时，`@InjectMocks`注入的mock对象就不生效了，因为这时候`UserService`实例是动态代理对象，不是`UserServiceImpl`。

如果aop是jdk动态代理，这时候代理类里没有目标类`UserService`的任何属性，mock注入自然不成功。

如果aop是cglib，代理类继承了目标类，包含目标类的属性，分属性有setter和无setter情况。

1. 如果属性有setter方法，那么注入时会使用setter，同时setter也是被代理的，所以mock能注入到目标类，mock生效。
2. 如果属性无setter，那么会注入到代理类，mock不生效。

不论是jdk动态代理还是cglib，都需要目标类被注入mock，mock才能生效。有两种方式。

方式一，只适用cglib，给目标类的属性加上setter方法。

方法二，两种aop通用，示例如下。

建议使用方法二，比较通用，且cglib加setter需要改动源码。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-config.xml")
public class AopUserServiceTest {
    @Mock
    private UserDao userDao;

    @Autowired
    private AopUserService aopUserService;

    @Before
    public void before() {
        MockitoAnnotations.openMocks(this);
      // 获取目标类，对目标类的属性进行注入
        AopUserService target = AopTestUtils.getTargetObject(aopUserService);
        ReflectionTestUtils.setField(target, "userDao", userDao);
    }

    @Test
    public void getUser() {
        Mockito.when(userDao.getUser()).thenReturn(User.builder().name("kk").age(20).build());

        System.out.println(aopUserService.getUser());
    }
}
```





## 参考文档

[Mock代理对象失效问题分析](https://www.codenong.com/cs109464967/)

[使用强大的 Mockito 测试框架来测试你的代码](https://juejin.cn/post/6844903439370043400)
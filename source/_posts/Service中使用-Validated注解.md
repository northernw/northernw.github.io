---
title: Service中使用@Validated注解
tags:
  - null
categories:
  - null
date: 2019-11-06 16:32:52
---



网上找到的资料大多介绍在`controller`中的使用，在工作中呢，往往需要在`service`中做校验，便想着是否可以实现呢。

顺着`@Validated`在`controller`中起作用的源码，可以发现校验的工作是交给了`LocalValidatorFactoryBean`，是spring做的封装接口，实现类是hibernate的`ValidatorImpl`.

也可以直接用hibernate的`ValidatorImpl`的，单纯的进行validate.

不严谨的性能比较：

1. 比手工校验不用说是比较慢了
2. 盲猜有部分缓存实现，validate的第一次校验比较慢，之后的校验触发比较快，大约是100倍
3. 比较匆忙，都是相同的入参出参、同一个方法，没有更多横向比较



![image-20191106164316937](/github/northernw.github.io/image/image-20191106164316937.png)



在配置文件中注入bean：

```xml
<bean id="validator"
          class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
        <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
    </bean>
```



业务代码示例：

```java
@Slf4j
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private LocalValidatorFactoryBean localValidatorFactoryBean;

    @Override
    public Boolean add(User user) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        DataBinder dataBinder = new DataBinder(user);
        dataBinder.addValidators(localValidatorFactoryBean);
//        dataBinder.validate();
        dataBinder.validate(User.Insert.class);
        BindingResult bindingResult = dataBinder.getBindingResult();
        log.info("validate(), error size = {}", bindingResult.getAllErrors().size());
        stopWatch.stop();
        log.info("cost time = {}", stopWatch.getLastTaskTimeNanos());
//        if (bindingResult.hasErrors()) {
//            log.info("errors = {}", bindingResult.getAllErrors());
//            throw new IllegalArgumentException(bindingResult.getAllErrors().get(0).getDefaultMessage());
//        }
        try {
            stopWatch.start();
            String name = user.getName();
            Assert.notNull(name, "not null");
            Integer age = user.getAge();
            Assert.notNull(age, "年龄不能为空");
        } catch (RuntimeException e) {
            stopWatch.stop();
            log.info("cost time 2 = {}", stopWatch.getLastTaskTimeNanos());
        }
        return user != null;
    }
}
```


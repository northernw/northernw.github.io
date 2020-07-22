---
title: 对SpringIOC循环依赖的理解
tags:
  - null
categories:
  - null
date: 2020-06-18 21:39:03
---



spring里有三级缓存，这三个map更多的是标识bean的创建状态

singletonObjects “单例池” “容器”，缓存创建完成的单例bean的地方

earlySingletonObjects 早期的bean，不是完整的，只是一个instance

singletonFactories 创建bean的原始工厂（实例化完成createBeanInstance，populateBean和initializeBean还没执行，放的是原始工厂。提前暴露的factory，在getObject的时候会执行SmartInstantiationAwareBeanPostProcessor接口的BeanPostProcessor，也包括动态代理的那些，能保证取到的bean引用是最终的bean）



**earlySingletonObjects和singletonFactories没有必然的联系，不循环依赖的场景也要走singletonFactories的**
    

```java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```
```java
	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 先看singletonObjects
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
        // 再看earlySingletonObjects
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
          // 如果允许earlyReference，从singletonFactories中取bean
          // singletonFactories中的bean是经过getEarlyBeanReference的bean，一大作用是提前创建好代理类，放入的是代理类，作用就是为了处理循环依赖
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return (singletonObject != NULL_OBJECT ? singletonObject : null);
	}

	/**
	 * Obtain a reference for early access to the specified bean,
	 * typically for the purpose of resolving a circular reference.
	 * @param beanName the name of the bean (for error handling purposes)
	 * @param mbd the merged bean definition for the bean
	 * @param bean the raw bean instance
	 * @return the object to expose as bean reference
	 */
	protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```





ioc

AbstractApplicationContext#refresh#finishBeanFactoryInitialization

DefaultListableBeanFactory#preInstantSingletons

AbstractBeanFactory#getBean#doGetBean

DefaultSingletonBeanRegistry#getSingleton

AbstractautowireCapableBeanFactory#createBean#doCreateBean

`createBeanInstance` -- 实例化对象，反射或cglib

`populateBean` -- 属性注入

`initializeBean` -- 执行工厂回调，初始化方法init-method和bean后置处理BeanPostProcessor（代理就是在这里创建的）

![image-20200618192017006](/github/northernw.github.io/image/image-20200618192017006.png)


​        

ABC循环依赖实例

![image-20200618205728132](/github/northernw.github.io/image/image-20200618205728132.png)

到C取A的引用时，ObjectFactory的getObject调用了BBP的一些处理getEarlyBeanReference（例如动态代理的创建）

C引用到的是A的最终bean

对动态代理（动态代理里的是A的实例）：C取完对A的引用，B取完对C的引用，A取完对B的引用，A继续注入其他的属性，米有毛病

对cglib：C取完对A的引用，这时A的实例是一个新的cglib产生的，容器中最终A的实例是这个cglib产生的，同时，**A的B没有自动注入** cglib代理的类不能实现自动注入？--- 整明白了。虽然cglib生成的代理里bService属性是null，但是它持有一个AServiceImpl的引用！所以对功能没有影响。也别想着整个幺蛾子了！

![image-20200618205812481](/github/northernw.github.io/image/image-20200618205812481.png)

cglib的方式，正在创建的A实例还是当前的，earlySingleObject中有一个A的实例是C的属性注入过程中产生的。

![image-20200618211703596](/github/northernw.github.io/image/image-20200618211703596.png)

![image-20200618211857154](/github/northernw.github.io/image/image-20200618211857154.png)

![image-20200618212929484](/github/northernw.github.io/image/image-20200618212929484.png)

![image-20200618213655441](/github/northernw.github.io/image/image-20200618213655441.png)
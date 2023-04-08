原文：

[Spring源码初探-IOC(4)-Bean的初始化-循环依赖的解决](https://www.jianshu.com/p/6c359768b1dc)



## 前言

在实际工作中，经常由于设计不佳或者各种因素，导致类之间相互依赖。这些类可能单独使用时不会出问题，但是在使用Spring进行管理的时候可能就会抛出BeanCurrentlyInCreationException等异常 。当抛出这种异常时表示Spring解决不了该循环依赖，本文将简要说明Spring对于循环依赖的解决方法。

## 循环依赖的产生和解决的前提

循环依赖的产生可能有很多种情况，例如：

- A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象
- A的构造方法中依赖了B的实例对象，同时B的某个field或者setter需要A的实例对象，以及反之
- A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象，以及反之

当然，Spring对于循环依赖的解决不是无条件的，首先前提条件是针对scope单例并且没有显式指明不需要解决循环依赖的对象，而且要求该对象没有被代理过。同时Spring解决循环依赖也不是万能，以上三种情况只能解决两种，第一种在构造方法中相互依赖的情况Spring也无力回天。结论先给在这，下面来看看Spring的解决方法，知道了解决方案就能明白为啥第一种情况无法解决了。

## Spring对于循环依赖的解决

Spring循环依赖的理论依据其实是Java基于引用传递，当我们获取到对象的引用时，对象的field或者或属性是可以延后设置的。
Spring单例对象的初始化其实可以分为三步：

- createBeanInstance， 实例化，实际上就是调用对应的构造方法构造对象，此时只是调用了构造方法，spring xml中指定的property并没有进行populate
- populateBean，填充属性，这步对spring xml中指定的property进行populate
- initializeBean，调用spring xml中指定的init方法，或者AfterPropertiesSet方法
  会发生循环依赖的步骤集中在第一步和第二步。

### 三级缓存

对于单例对象来说，在Spring的整个容器的生命周期内，有且只存在一个对象，很容易想到这个对象应该存在Cache中，Spring大量运用了Cache的手段，在循环依赖问题的解决过程中甚至使用了“三级缓存”。

“三级缓存”主要是指

```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
```

从字面意思来说：singletonObjects指单例对象的cache，singletonFactories指单例对象工厂的cache，earlySingletonObjects指提前曝光的单例对象的cache。以上三个cache构成了三级缓存，Spring就用这三级缓存巧妙的解决了循环依赖问题。

### 解决方法

回想上篇文章中关于Bean创建的过程，首先Spring会尝试从缓存中获取，这个缓存就是指singletonObjects，主要调用的方法是：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
   Object singletonObject = this.singletonObjects.get(beanName);
   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
      synchronized (this.singletonObjects) {
         singletonObject = this.earlySingletonObjects.get(beanName);
         if (singletonObject == null && allowEarlyReference) {
            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
            if (singletonFactory != null) {
               singletonObject = singletonFactory.getObject();
               this.earlySingletonObjects.put(beanName, singletonObject);
               this.singletonFactories.remove(beanName);
            }
         }
      }
   }
   return (singletonObject != NULL_OBJECT ? singletonObject : null);}
```

首先解释两个参数：

- isSingletonCurrentlyInCreation 判断对应的单例对象是否在创建中，当单例对象没有被初始化完全(例如A定义的构造函数依赖了B对象，得先去创建B对象，或者在populatebean过程中依赖了B对象，得先去创建B对象，此时A处于创建中)
- allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象

分析getSingleton的整个过程，Spring首先从singletonObjects（一级缓存）中尝试获取，如果获取不到并且对象在创建中，则尝试从earlySingletonObjects(二级缓存)中获取，如果还是获取不到并且允许从singletonFactories通过getObject获取，则通过singletonFactory.getObject()(三级缓存)获取。如果获取到了则

```java
this.earlySingletonObjects.put(beanName, singletonObject);
this.singletonFactories.remove(beanName);
```

则移除对应的singletonFactory,将singletonObject放入到earlySingletonObjects，其实就是将三级缓存提升到二级缓存中！

Spring解决循环依赖的诀窍就在于singletonFactories这个cache，这个cache中存的是类型为ObjectFactory，其定义如下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;}
```

在bean创建过程中，有两处比较重要的匿名内部类实现了该接口。一处是

```java
new ObjectFactory<Object>() {
   @Override   public Object getObject() throws BeansException {
      try {
         return createBean(beanName, mbd, args);
      }      catch (BeansException ex) {
         destroySingleton(beanName);
         throw ex;
      }   }
```

在上文已经提到，Spring利用其创建bean（这样做真的很不明确呀...）

另一处就是:

```java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   public Object getObject() throws BeansException {
      return getEarlyBeanReference(beanName, mbd, bean);
   }});
```

此处就是解决循环依赖的关键，这段代码发生在createBeanInstance之后，也就是说单例对象此时已经被创建出来的。这个对象已经被生产出来了，虽然还不完美（还没有进行初始化的第二步和第三步），但是已经能被人认出来了（根据对象引用能定位到堆中的对象），所以Spring此时将这个对象提前曝光出来让大家认识，让大家使用。

这样做有什么好处呢？让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，长大成人，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象也蜕变完美了！一切都是这么神奇！！

知道了这个原理时候，肯定就知道为啥Spring不能解决“A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象”这类问题了！

## 总结

Spring通过三级缓存加上“提前曝光”机制，配合Java的对象引用原理，比较完美地解决了某些情况下的循环依赖问题！

作者：LNAmp
https://www.jianshu.com/p/6c359768b1dc来源：简书著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
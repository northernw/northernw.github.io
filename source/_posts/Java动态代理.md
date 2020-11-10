---
title: Java动态代理
tags:
  - KNOWLEDGE
  - Java
  - 动态代理
categories:
  - KNOWLEDGE
date: 2020-11-09 16:15:26
---

###### 小结

1. 实现一个调用处理器InvocationHandler h，并以某种方式持有被代理实例，在invoke方法中，触发被代理实例的方法
2. `Proxy.newProxyInstance(..)`需要传入接口的类加载器、接口类以及调用处理器，生成一个代理实例，所以呢
   1. 代理实例持有h，h持有被代理实例
   2. 调用代理实例的x方法，会触发h的invoke方法，再触发被代理实例的x方法



###### Spring中对动态代理的应用

JdkDynamicAopProxy.java

1. 它是一个InvocationHandler实例
2. 持有一个AdvisedSupport，AdvisedSupport中有被代理实例、被代理实例的接口们、Advisors增强or切面
3. invoke方法中，会判断当前方法是否有命中切面，执行有切面（责任链模式）or无切面的逻辑，触发被代理实例的方法



###### Java动态代理的使用

接口类

```java
public interface UserService {
    String getName();
}
```

实现类

```java
public class UserServiceImpl implements UserService {
    @Override
    public String getName() {
        System.out.println("in method: getName");
        return "KK";
    }
}
```

InvocationHadler

```java
public class LogInvocationHandler<T> implements InvocationHandler {
    T target;

    public LogInvocationHandler(T target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before proxy...");
        Object result = method.invoke(target, args);
        System.out.println("after proxy...");
        return result;
    }
}
```

客户端

```java
public class ProxyMain {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        LogInvocationHandler handler = new LogInvocationHandler<>(userService);
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(UserService.class.getClassLoader(), new Class<?>[]{UserService.class}, handler);
        System.out.println(userServiceProxy.getName());
    }
}
```



###### 原理说明

关键点在于`Proxy.newProxyInstance(UserService.class.getClassLoader(), new Class<?>[]{UserService.class}, handler);`，生成了动态代理类。

Proxy的源码：

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
  // 这里生成了代理类的字节码，并加载进虚拟机
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

          // 代理类的构造器
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
          // 创建代理类的实例
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

如何生成代理类的字节码呢？





再看下生成的代理类字节码什么样子。

```java
        byte[] classFile = ProxyGenerator.generateProxyClass("$Proxy0", new Class<?>[]{UserService.class});
        String filename = "UserServiceProxy.class";
        try (FileOutputStream fos = new FileOutputStream(filename)) {
            fos.write(classFile);
            fos.flush();
            System.out.println("done");
        } catch (Exception e) {
            e.printStackTrace();
        }
```

`UserServiceProxy.class`反编译后：

```java
public final class $Proxy0 extends Proxy implements UserService {
  //第二步, 生成静态域
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

  //第一步, 生成构造器
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

  //第三步, 生成代理方法
    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String getName() throws  {
        try {
            return (String)super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

  //第四步, 生成静态初始化方法
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("wyq.learning.quickstart.proxy.UserService").getMethod("getName");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```


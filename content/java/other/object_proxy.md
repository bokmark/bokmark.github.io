---
title: '动态代理'
date: 2018-11-28T15:14:39+10:00
weight: 4
summary: "Java的动态代理"
---
> 在研究retrofit源码的时候，使用了动态代理的技术。顿时让我觉得这个技术很神奇，那么我们就来看看到底动态代理是怎么实现的。
> 动态代理相对于静态代理来说，动态代理对象调用的每一个方法都能回调到InvocationHandler的回调中。

## 使用
```
Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[] {service},
            new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
                  throws Throwable {
                // If the method is a method from Object then defer to normal invocation.
                if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
                }
                args = args != null ? args : emptyArgs;
                return platform.isDefaultMethod(method)
                    ? platform.invokeDefaultMethod(method, service, proxy, args)
                    : loadServiceMethod(method).invoke(args);
              }
            });
```

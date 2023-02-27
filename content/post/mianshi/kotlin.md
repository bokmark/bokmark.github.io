---
author: Bokmark Ma
title: "各种面试的答案 - Kotlin"
date: 2022-10-16T00:35:27+08:00
description: 面试题 kotlin部分答案
slug: m/kotlin
categories:
    - Mianshi
tags:
    - 面试
    - Kotlin
---

# 各种面试的答案 - Kotlin
 
## Object 和 Companion Object 的区别。

> object 可用于快速定义单例以及匿名类。而companion object 定义为伴生对象，具有与class相同的伴生属性，就相当于Java中的静态内部类。  

*object如何快速定义单例？先看代码：*
```kotlin
// kotlin source code
object Single {}
// java decompile code 
public final class Single {
   @NotNull
   public static final Single INSTANCE;

   private Single() {
   }

   static {
      Single var0 = new Single();
      INSTANCE = var0;
   }
}
```
从上面我们可以看到，object相当于定义了一个静态内部属性 INSTANCE，再在静态代码块里实现赋值操作。  

*object如何实现匿名内部类?先看代码：*
```kotlin
// kotlin source code
class Single {
    val a = object {
        val b = 1
    }
}
// java decompile code 
public final class Single {
   @NotNull
   private final Object a = new Object() {
      private final int b = 1;

      public final int getB() {
         return this.b;
      }
   };

   @NotNull
   public final Object getA() {
      return this.a;
   }
}
```
和Java的匿名内部类实现类似。  

*Companion object 如何实现伴生?先看代码：*
```kotlin
// kotlin source code
class Single {
    companion object {
        val a = 1
    }
}
// java decompile code

public final class Single {
   private static final int a = 1;
   @NotNull
   public static final Companion Companion = new Companion((DefaultConstructorMarker)null);
 
   public static final class Companion {
      public final int getA() {
         return Single.a;
      }

      private Companion() {
      }
      // 这个方法 是匿名内部类都会由系统自动生成的一个方法。
      // 外部类持有匿名内部类的一个引用，但是匿名内部类只有一个private的构造方法，所以需要一个其他的构造方法来构建。
      // 那么包含一个隐藏起来对外不可见的类的参数的构造方法就出现了。如下：
      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```
具体实现就是实现了一个静态内部类，默认名字为Companion。
 
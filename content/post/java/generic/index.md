---
title: "泛型"
date: 2023-03-09T23:32:12+08:00
description: 一篇文章说透泛型
draft: true
slug: generic_type
---

# 泛型

## 什么是泛型？
> 本菜鸡一直认为真正想要了解一个人，就要了解TA的过去，东西也一样。  
> 所以理解什么是泛型就要理解泛型是怎么来的。 

泛型在JDK 1.5的版本才被引入，那么在1.5以前程序员们是怎么处理以下这段代码的呢？
```java
List<Integer> nums = new ArrayList<>();
for (int i = 0; i < 100; i++) {
    nums.add(i);
}
int sum = 0;
for (int i = 0; i < 100; i++) {
    sum += nums.get(i)
}
print(sum);
```
这段代码非常简单，就是往list里面加入100个数，然后把这些数字都加起来看起来非常简洁，但是在1.5以前程序员们是这么处理的
```java
List nums = new ArrayList();
for (int i = 0; i < 100; i++) {
    nums.add((Integer) i);
}
int sum = 0;
for (int i = 0; i < 100; i++) {
    sum += (int) nums.get(i);
}
print(sum);
```
你会发现，1.5以前多了很多的强制转化。
而做得多就错得多，这种写法不简洁而且容易出问题，所以泛型乘势出击。

## 泛型的擦除
泛型是一种很早就出现的编程思想与其他类C的语法不同，JAVA的泛型是虚拟的泛型。  
- 为什么是虚拟的泛型，而不是真实的泛型？

因为泛型是JAVA开发过程中加入的，必须要兼容嘛！总不能以前的老代码不要了吧，那以前的程序员会掀桌子的。  
但是泛型真的很有用呀，怎么办？假的泛型也是泛型呀！  
没有枪头的枪也可以捅人的呀！  

那么为了实现假泛型，JAVA的做法是，泛型只在编译期有用，到运行的时候，直接擦除你的泛型类型！哎就是玩！  
为了不做得那么绝，泛型的类型还是会保存在原始数据中的。

## 什么地方能用泛型
在定义类、接口和方法时，可以附带类型参数，使其变成泛型类、泛型接口和泛型方法。与非泛型代码相比，使用泛型有三大优点：更健壮（在编译时进行更强的类型检查）、更简洁（消除强转，编译后自动会增加强转）、更通用（代码可适用于多种类型）

泛型本质上是 Javac 编译器的一颗 语法糖，这是因为：泛型是 JDK1.5 中引进的新特性，为了 向下兼容，Java 虚拟机和 Class 文件并没有提供泛型的支持，而是让编译器擦除 Code 属性中所有的泛型信息，需要注意的是，泛型信息会保留在类常量池的属性中。

类型擦除发生在编译时，具体分为以下 3 个步骤：

- 擦除所有类型参数信息，如果类型参数是有界的，则将每个参数替换为其第一个边界；如果类型参数是无界的，则将其替换为 Object
- 必要时）插入类型转换，以保持类型安全
- 必要时）生成桥接方法以在子类中保留多态性 

## 通配符
通配符是什么？
专门用于批量处理集合时，才会使用的内容。

### 当通配符遇到Kotlin
有人会问Java中的`<? extends T>` 和 Kotlin中的`<out T>` 有什么区别？
如果用于约束集合是没有什么区别的，完全可以等效看待。
但是Kotlin 可以这么定义类
```kotlin
class G<T> {
    fun get(): T {
        return null!!
    }

    fun put(t: T) {

    }
}
// in T只能写在方法的参数里面（in），对于调用G2的人说是输入T的，所以是消费者
class G1<in T> {
    fun get(): @UnsafeVariance T {
        return null!!
    }

    fun put(t: T) {

    }
}
// out T只能写在方法的参数的外面(out)，对于调用G2的人说是提供T的，所以是生产者
class G2<out T> {
    fun get(): T {
        return null!!
    }

    fun put(t: @UnsafeVariance T) {

    }
}
// PECS Producter extends consumer super

```

## 参考资料
- [关于泛型能问的都在这里了](https://juejin.cn/post/6888345234653052941)
- [通配符](https://developer.aliyun.com/article/640124)
- [Kotlin泛型](https://www.kotlincn.net/docs/reference/generics.html)
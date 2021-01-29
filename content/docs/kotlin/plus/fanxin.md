---
title: 'Kotlin 泛型'
date: 2020-11-30T19:30:08+10:00
draft: false
weight: 2
---

### java的泛型
> kotin的泛型类型 与java一样中几乎一致 没有什么大的区别，相应的例子也在[这里](/kotlin/intro/) 有体现。
> [官方介绍](https://www.kotlincn.net/docs/reference/generics.html)

##### java 泛型中的通配符
> 首先需要和各位读者明确java中的泛型的两个概念，这有助于你理解kotlin的泛型

- 泛型定义，上限和下限
- 泛型读取模式

1. 泛型定义是指 在类或者方法上的定义的泛型，这些定义我们 可以定义上限和下限。比如这段代码
  ```java
    class A{}
    class B extends A{}
    class C extends A{}
    class Fu<T> {}
    class Zi<T extends B> extends Fu<T> {}
  ```
  Zi class定义了他的泛型类型具有上限是B，意味着他的类型只能传入B的子类。

  你不能在Java中这样定义 `class A<? extends T>` 因为这样编译器无法在编译阶段确定他的类型

2. 泛型读取模式

  ```java
    List<? extends A> list = new ArrayList<B>();
    A a0 = list.get(0);
    list.add(new A());
  ```
  这一段代码中`list.add(new A());` 会报错。这是因为当我们在使用？extends xx的时候会有一个规则就是读写权限限制。我们再来看一段代码：
  ```Java
    List<? super B> list = new ArrayList<A>();
    B b = list.get(0);
    list.add(new B());
  ```
  这一段代码中`B b = list.get(0);` 会报错。
  其实我们去读写权限安全的角度去思考编译器的处理逻辑就能够理解。
  - 当我们声明一个变量为 `? extends A` 的时候，是希望这个泛型变量接受的泛型类型为A的子类，而且我们只关注A的方法，不考虑子类的扩展或者重载（不然直接声明一个子类的泛型变量即可）
  - 以list为例，当声明了一个list，泛型类型为A的子类，那么这个list的元素是不是可以有很多种子类的可能（比如B或者C）。这就会遇到一个问题，如果允许这个list可以修改，那么运行期间就可能报错，（比如arraylist是使用数组来实现的，但是你类型不同，数组怎么实现？）。所有为了避免这种问题，编译器所幸就拒绝你对其进行修改只允许获取。
  - 同理，当你声明一个变量为`? super B`的时候，你是希望这个泛型变量接收的泛型类型为B的父类，而且我们关注点在B的方法上面。同样以list为例，当你声明了一个list，泛型类型为B的父类那么这个list的元素的时候，你去获取这个元素的时候你得到的引用为B，但是这个B 可能是b的父类的元素，这中间就会产生转化错误（我们可以用父类去引用子类的实例，但是不能用子类去引用父类的实例）。那么编译器就索性拒绝你对其进行获取，但是允许你修改,当然你只可以添加B的类型的实例。

3. 两者的区别

  像第一小节所的你不能在Java中这样定义 `class A<? extends T>`。这就导致了Java中泛型其实可以分为两种理解，第一种就是在定义的时候规定的泛型，我们叫他泛型定义。还有一种就是泛型读取模式定义了一个类型引用了一个变量，限制了对这个变量的各种读取模式。

### kotlin 的泛型
> *当你理解Java的泛型之后用`out`代替 `? extends`。用 `in`代替`? super` 就可以了。*

当然kotlin作为一门有野心的语音不会这么简单的处理泛型。那么有什么需要注意的嘛？接下来我们来看看
- 扩展到整个类的修饰
  ```kotlin
    class Array<T>(val size: Int) {
      fun get(index: Int): T { …… }
      fun set(index: Int, value: T) { …… }
    }
  ```
  这里当我们给一个类 定义了泛型的时候类中的各个方法和属性就可以使用这个T。那么我们这么定义呢
  ```Kotlin
    class Array<in T>(val size: Int) {
      fun get(index: Int): T { …… } // 这里会报错
      fun set(index: Int, value: T) { …… }
    }
  ```
  这里我们就可以发现类里面的所有T 都带上了in的限制。

  *这一点Java是做不到的，这个特性也是Kotlin 特有的*

- 使用处型变：类型投影
  ```
  fun copy(from: Array<out Any>, to: Array<Any>) { …… }
  ```
  在这里定义了out 就限制了from数组被修改。

- 星投影 （copy 从官网）

  有时你想说，你对类型参数一无所知，但仍然希望以安全的方式使用它。 这里的安全方式是定义泛型类型的这种投影，该泛型类型的每个具体实例化将是该投影的子类型。

  Kotlin 为此提供了所谓的星投影语法：

  - 对于 Foo <out T : TUpper>，其中 T 是一个具有上界 TUpper 的协变类型参数，Foo <*> 等价于 Foo <out TUpper>。 这意味着当 T 未知时，你可以安全地从 Foo <*> 读取 TUpper 的值。
  - 对于 Foo <in T>，其中 T 是一个逆变类型参数，Foo <*> 等价于 Foo <in Nothing>。 这意味着当 T 未知时，没有什么可以以安全的方式写入 Foo <*>。
  - 对于 Foo <T : TUpper>，其中 T 是一个具有上界 TUpper 的不型变类型参数，Foo<*> 对于读取值时等价于 Foo<out TUpper> 而对于写值时等价于 Foo<in Nothing>。

  如果泛型类型具有多个类型参数，则每个类型参数都可以单独投影。 例如，如果类型被声明为 interface Function <in T, out U>，我们可以想象以下星投影：

  - Function<*, String> 表示 Function<in Nothing, String>；
  - Function<Int, *> 表示 Function<Int, out Any?>；
  - Function<*, *> 表示 Function<in Nothing, out Any?>。

  *注意：星投影非常像 Java 的原始类型，但是安全。*

- 泛型擦除

  这一点和Java 类似在虚拟机上泛型是没有具体的类型的。这些仅在编译器检查。is as 之类的关键字也不能再这一点上进行判断

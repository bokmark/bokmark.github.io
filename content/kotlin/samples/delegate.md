---
title: '【译】Kotlin 语法基础 - 代理'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: '委托模式 、 代理属性 、 lazy、to'
---

### Delegation Pattern 委托模式

kotlin 天然支持 [委托代理模式](https://kotlinlang.org/docs/reference/delegation.html)，不再需要编写一些模板文件。

```
interface SoundBehavior {                                                          // 1
    fun makeSound()
}

class ScreamBehavior(val n:String): SoundBehavior {                                // 2
    override fun makeSound() = println("${n.toUpperCase()} !!!")
}

class RockAndRollBehavior(val n:String): SoundBehavior {                           // 2
    override fun makeSound() = println("I'm The King of Rock 'N' Roll: $n")
}

// Tom Araya is the "singer" of Slayer
class TomAraya(n:String): SoundBehavior by ScreamBehavior(n)                       // 3

// You should know ;)
class ElvisPresley(n:String): SoundBehavior by RockAndRollBehavior(n)              // 3

fun main() {
    val tomAraya = TomAraya("Thrash Metal")
    tomAraya.makeSound()                                                           // 4
    val elvisPresley = ElvisPresley("Dancin' to the Jailhouse Rock.")
    elvisPresley.makeSound()
}
/*
THRASH METAL !!!
I'm The King of Rock 'N' Roll: Dancin' to the Jailhouse Rock.
*/
```

- 1 定义一个接口
- 2 定义两个实现类，`ScreamBehavior` 和` RockAndRollBehavior`。他们都实现了 `SoundBehavior`的接口。
- 3 `TomAraya ` 和`ElvisPresley ` 也是先了`SoundBehavior`的接口，但是他们没有提供实现，而是将实现委托给了`ScreamBehavior` 和` RockAndRollBehavior`。通过一个`by` 关键字，这个委托就产生了，你看没有一丝一毫的多余代码。
- 4 当调用了 tomAraya 的 ` makeSound()`  或者嗲用 elvisPresley 的 ` makeSound()`的时候，他们分别调用了他们的委托类。

___

### Delegated Properties 代理属性

kotlin 提供了一种[代理属性](http://kotlinlang.org/docs/reference/delegated-properties.html)的机制,这个机制允许你将一个属性的get set 方法委托给某一个实例，这个实例必须有`getValue` 方法，如果待委托的属性还是一个可变属性那么 `setValue` 也是必须的。

```
import kotlin.reflect.KProperty

class Example {
    var p: String by Delegate()                                               // 1

    override fun toString() = "Example Class"
}

class Delegate() {
    operator fun getValue(thisRef: Any?, prop: KProperty<*>): String {        // 2     
        return "$thisRef, thank you for delegating '${prop.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, prop: KProperty<*>, value: String) { // 2
        println("$value has been assigned to ${prop.name} in $thisRef")
    }
}

fun main() {
    val e = Example()
    println(e.p)
    e.p = "NEW"
}
/*
Example Class, thank you for delegating 'p' to me!
NEW has been assigned to p in Example Class
*/
```

- 1 委托属性 `p` 类型是`String` 被委托给了一个class `Delegate`的实例，这个实例用`by`关键字定义在 p之后。
- 2 委托类里面的委托方法，这个方法的签名必须和这里一摸一样。方法里面你可以由自己决定需要几步操作。如果是不可变属性那么只有getValue 是必须的。

##### 标准库里的delegate

kotlin的标准库里面提供了一大堆有用的delegate，比如 `lazy` `observable` 等等。比如`lazy`的使用方法。

```
class LazySample {
    init {
      println("created!")            // 1
    }

    val lazyStr: String by lazy {
        println("computed!")          // 2
        "my lazy"
    }
}

fun main() {
    val sample = LazySample()         // 1
    println("lazyStr = ${sample.lazyStr}")  // 2
    println(" = ${sample.lazyStr}")  // 3
}
/*
created!
computed!
lazyStr = my lazy
 = my lazy
*/
```

- 1 lazy 的属性不会在类实例创建的时候初始化
- 2 第一次调用`lazyStr`的`get()`的时候，才会调用lazy类的实例的提供的表达式，返回一个"my lazy"。这个mylazy将会被赋值给lazyStr
- 3 再次调用get的时候将会直接返回之前储存的值。

*如果你想要一个线程安全的 延迟初始化方案，请使用 `blockingLazy()` 代替。他保证了 getValue 后者 setValue 会只在一个线程上运行，并且所有的线程都可以读取到同样的值。*

### 保存属性到 map

属性委托，同样可以用于保存内容到map 。这是很方便地用于类似于解析json，或者其他的什么东西之上的。

```
class User(val map: Map<String, Any?>) {
    val name: String by map                // 1
    val age: Int     by map                // 1
}

fun main() {
    val user = User(mapOf(
            "name" to "John Doe",
            "age"  to 25
    ))

    println("name = ${user.name}, age = ${user.age}")
}
//name = John Doe, age = 25
```

- 1 在这里 map 内部提供过来 delegate 属性类的作用，当调用user的属性的get的时候，会去map中取出value，key是user属性的名字。

*当然你也可以用可变属性 不需要一定要val 的属性，只是你需要注意的是 如果你定义的是可变属性，那么你传入的map 也必须是可变map*

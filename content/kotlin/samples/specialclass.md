---
title: '【译】Kotlin 语法基础 - 特殊类'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: 'Data class、enum class、sealed class、object keyword'
---


### Data Class

[Data class](https://kotlinlang.org/docs/reference/data-classes.html?_ga=2.47914596.2133596250.1603679551-1735323035.1603280899) 可以用于创建一个数据类用于存储数据。这些类将会自动提供类似`setter`, `getter`,`copy`,`toString()`,`hashCode()`,`componentN()`方法

```
data class User(val name: String, val id: Int)             // 1

fun main() {
    val user = User("Alex", 1)
    println(user)                                          // 2

    val secondUser = User("Alex", 1)
    val thirdUser = User("Max", 2)

    println("user == secondUser: ${user == secondUser}")   // 3
    println("user == thirdUser: ${user == thirdUser}")

    println(user.hashCode())  
    println(secondUser.hashCode())                               // 4
    println(thirdUser.hashCode())

    // copy() function
    println(user.copy())                                   // 5
    println(user.copy("Max"))                              // 6
    println(user.copy(id = 2))                             // 7

    println("name = ${user.component1()}")                 // 8
    println("id = ${user.component2()}")
}
/*
User(name=Alex, id=1)
user == secondUser: true
user == thirdUser: false
63347075
63347075
2390846
User(name=Alex, id=1)
User(name=Max, id=1)
User(name=Alex, id=2)
name = Alex
id = 1
*/
```

- 1 定义一个data class
- 2 `println` 输出 dataclass 的 `toString()` 方法的返回值
- 3 自动编码的`equals()` 方法将会考虑如果双方实例所有属性全部相等，那么者两个实例将会相等
- 4 相等的 dataclass hashcode 在equals的情况下也会返回相同的hashcode
- 5 自动编码的`copy()` 方法将会简便的创建一份副本
- 6 当你选择拷贝的时候，你可以便捷的修改内部属性。你可以按照构造函数的顺序依次传入相应的值
- 7 你也可以通过命名参数的方式来修改某个特定的参数
- 8 自动编码的`componentN()` 方法，可以让你以定义属性的顺序来依次地读取各个字段的值

### Enum class 枚举

[枚举类](https://kotlinlang.org/docs/reference/enum-classes.html) 等同于java中地枚举

```
enum class State {
    IDLE, RUNNING, FINISHED                           // 1
}

fun main() {
    val state = State.RUNNING                         // 2
    val message = when (state) {                      // 3
        State.IDLE -> "It's idle"
        State.RUNNING -> "It's running"
        State.FINISHED -> "It's finished"
    }
    println(message)
}
```

- 1 定义具有三个实例地枚举类，每个实例都是唯一且固定地，枚举的数量也是固定不可变的。
- 2 可以访问一个枚举实例通过他的类名。
- 3 本俩当when 当作表达式的时候是需要明确使用else 字段的。但是当when 表达式遇到 枚举的时候，只要你列举了所有的额枚举变量，那么你就不需要else 字段了

枚举类，也可以像Java 那样在类里面定义方法字段等。这种情况你需要明确的使用分号来分割 枚举的实例和类里的其他要素。

```
enum class Color(val rgb: Int) {                      // 1
    RED(0xFF0000),                                    // 2
    GREEN(0x00FF00),
    BLUE(0x0000FF),
    YELLOW(0xFFFF00);

    fun containsRed() = (this.rgb and 0xFF0000 != 0)  // 3
}

fun main() {
    val red = Color.RED
    println(red)                                      // 4
    println(red.containsRed())                        // 5
    println(Color.BLUE.containsRed())                 // 6
}
```

- 1 定义一个枚举类，具有一个属性和一个方法
- 2 每个实例，都必须传递一个明确的不相同的属性值
- 3 方法需要用分号和实力定义分隔开
- 4 默认生成的`toString` 方法，默认使用枚举的名字，比如这里将会打印出 `RED`。
- 5 这里会调用当前实例的`containsRed` 方法
- 6 你也可以通过类名来读取实例，然后再调用他的方法

### 密封类 Sealed Classes

[密封类](https://kotlinlang.org/docs/reference/sealed-classes.html) 密封类用于限制继承。如果你定义了一个密封类，那么他的子类必须定义在同一个文件中。不允许子类定义在别的文件中，

```
sealed class Mammal(val name: String)                                                   // 1

class Cat(val catName: String) : Mammal(catName)                                        // 2
class Human(val humanName: String, val job: String) : Mammal(humanName)

fun greetMammal(mammal: Mammal): String {
    when (mammal) {                                                                     // 3
        is Human -> return "Hello ${mammal.name}; You're working as a ${mammal.job}"    // 4
        is Cat -> return "Hello ${mammal.name}"    //5
        is Mammal -> return "Hello ${mammal.name}"   
    }                                                                                 //6
}

fun main() {
    println(greetMammal(Cat("Snowy")))  
    println(greetMammal(Human("John", "IT")))  
    //println(greetMammal(Mammal("John")))  // 7
}
/*
Hello Snowy
Hello John; You're working as a IT
*/
```

- 1 定义一个密封类
- 2 定义一个他的子类，这个子类必须定义在同一个文件中
- 3 在这个方法中，when 表达式 不需要定义 else 的字段，是因为穷举了 所有的可能性，这里有人会问了Mammal 不算嘛，其实是因为你创建不出来 Mammal 这个类，*7*  这里会报错 `Cannot access '<init>': it is private in 'Mammal'
  Sealed types cannot be instantiated`
- 4 这里将会发生一个快速的强制转化，将 Mammal  转化为 Human
- 5 这里将会发生一个快速的强制转化，将 Mammal  转化为 Cat
- 6 这里不需要else 字段，因为所有的 子类都以及明确地覆盖了
- 7 这句话将会报错

---

### Object Keyword

类和对象在kotlin中和别的面向对象的语言中大同小异：class 是一个蓝图，object 是类的一个实例。通常来说，你定义了一个类，然后创建多个类的实例。

```
import java.util.Random

class LuckDispatcher {                    //1
    fun getNumber() {                     //2
        var objRandom = Random()
        println(objRandom.nextInt(90))
    }
}

fun main() {
    val d1 = LuckDispatcher()             //3
    val d2 = LuckDispatcher()

    d1.getNumber()                        //4
    d2.getNumber()
}
```

- 1 定义一个蓝图
- 2 定义一个方法
- 3 创建一个蓝图的实例
- 4 调用实例的方法

在kotlin中中你有这么一个定义[object keyword]([https://kotlinlang.org/docs/reference/object-declarations.html](https://kotlinlang.org/docs/reference/object-declarations.html)
)。这常常被用来定义 一个唯一的实现。这在Java中，你可以理解为单例模式：当有两个线程想要创建这个类的实例的时候，能保证只有同时只有一个实例。

> 在kotlin中，你只需要申明一个 `object ：` 没有类定义，没有构造方法，只有一个lazy 的实例。why lazy？因为他只会在被访问到的时候才会被创建，另一方面来说：他甚至可能不会被创建

##### `object` Expression

这是一个`object` Expression 基础的用法,：一个简单的object或者数据的结构体。在这里你不需要定义一个类的申明：你只需要创建一个object然后申明他的成员，并且在一个方法中去使用他。想这样的对象，在Java中会以匿名内部类的实例存在。

```
fun rentPrice(standardDays: Int, festivityDays: Int, specialDays: Int): Unit {  //1

    val dayRates = object {                                                     //2
        var standard: Int = 30 * standardDays
        var festivity: Int = 50 * festivityDays
        var special: Int = 100 * specialDays
    }

    val total = dayRates.standard + dayRates.festivity + dayRates.special       //3

    print("Total price: $$total")                                               //4

}

fun main() {
    rentPrice(10, 2, 1)                                                         //5
}
```

- 1 创建一个方法，带有三个 参数
- 2 创建一个object 并计算保存三个结果
- 3 访问这个object的结构体里面的值
- 4 打印结果
- 5 运行这个方法，只有在运行这个方法的时候这个object 才会被创建

##### `object` Declaration

你也可以用 object 申明，这不是一个表达式，不能用于赋值语句，你最好通过类名直接访问他的成员

```
object DoAuth {                                                 //1
    fun takeParams(username: String, password: String) {        //2
        println("input Auth parameters = $username:$password")
    }
}

fun main(){
    DoAuth.takeParams("foo", "qwerty")                          //3
}
```

- 1 创建一个object 的申明
- 2 在object 中定义一个方法
- 3 执行，在这里object 才会被真正的创建。

##### Companion Objects内部object

申明在类的内部的另一种object的用法是 Companion Objects。从语法上来讲，这更像是java中的静态方法。如果你想在kotlin中使用Companion Object。请使用包级别的类方法定义。

```
class BigBen {                                  //1
    companion object Bonger {                   //2
        fun getBongs(nTimes: Int) {             //3
            for (i in 1 .. nTimes) {
                print("BONG ")
            }
        }
    }
}

fun main() {
    BigBen.getBongs(12)                         //4
}
```

- 1 定义一个class
- 2 定义一个 companion ，显示的定义了名字
- 3 定义了一个companion object方法，
- 4 通过companion object 的类名直接 访问他的方法

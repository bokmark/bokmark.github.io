---
title: '【译】Kotlin 语法基础 - 特殊特性'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: '命名参数、string模板、解构声明、smart cast'
---

### Named Arguments

和其他语言（java， c++，之类的）不同的是kotlin提供了一种特性叫命名参数。既可以和别的语言一样按照定义的方法的参数顺序来传递参数，也可以通过名字来传递参数，这样就避免了错误的顺序会带来的错误以及可以更清晰地调用。有些错误是很难以通过编译器来检查出编译错误的。比如这样

```
fun format(userName: String, domain: String) = "$userName@$domain"

fun main() {
    println(format("mario", "example.com"))                         // 1
    println(format("domain.com", "username"))                       // 2
    println(format(userName = "foo", domain = "bar.com"))           // 3
    println(format(domain = "frog.com", userName = "pepe"))         // 4
}
```

- 1 传递了两个参数
- 2 同样也是传递了两个参数，但是在这里其实是发生了业务上的错误的。如果你使用命名参数就可以更清晰地发现这个问题。
- 3 这里就使用了命名参数，是不是很清晰呀！
- 4 命名参数可以随意交换顺序哦

___

### String Templates string模板

String 模板可以允许你在string被需要使用（比如println的时候string 被需要使用）的时候，将写在String 中的引用和表达式替换成真正的值。

```
fun main() {
    val greeting = "Kotliner"
    var i = 1
    var str = "Hello $i"
    i = 2
    println(str)   
    println("Hello $i")
    str = "Hello ${++i}"
    println("Hello $i")

    println("Hello $greeting")                  // 1
    println("Hello ${greeting.toUpperCase()}")  // 2
}
/*
1
2
3
Hello Kotliner
Hello KOTLINER
*/
```

- 1 打印一个在String 中的变量应用 用`$` 开头表示
- 2 打印一个在String 中的表达式, 用`$`开头，用大括号包起来。

___

### Destructuring Declarations 解构声明

[解构声明](https://kotlinlang.org/docs/reference/multi-declarations.html#destructuring-declarations) 可以让你在只需要一个实例的成员的时候，可以用简单的几句就获取到你想要的结果。

```
fun findMinMax(list: List<Int>): Pair<Int, Int> {
    // do the math
    return Pair(50, 100)
}

fun main() {
    val (x, y, z) = arrayOf(5, 10, 15)                              // 1

    println("$x, $y, $z")

    val map = mapOf("Alice" to 21, "Bob" to 25)
    for ((name, age) in map) {                                      // 2
        println("$name is $age years old")          
    }

    val (min, max) = findMinMax(listOf(100, 90, 50, 98, 76, 83))    // 3
    println("$min, $max")

}
/*
5, 10, 15
Alice is 21 years old
Bob is 25 years old
50, 100
*/
```

- 1 将array的 三个元素分别赋值给 x y z
- 2 将map的entry 分为 name 和 age 两个属性，在for循环中
- 3 这里的i情况其实和2 是差不多的 就是将pair 分为赋值给 min 和max。这里返回值也可以作为解构声明的一部分

```
data class User(val username: String, val email: String)    // 1

fun getUser() = User("Mary", "mary@somewhere.com")

fun main() {
    val user = getUser()
    val (username, email) = user                            // 2
    println(username == user.component1())                  // 3

    val (_, emailAddress) = getUser()                       // 4

}
```

- 1 定义一个data class
- 2 将user的 两个属性按照名字解构说明分给两个不可变变量
- 3 这里返回的是 true。这里通过自动编码加进去的 `componentN()` 方法来获取user的第几个属性。
- 4 解构声明也可以忽略某个属性，这里和go语言的多返回值很像。这里忽略的意义就是告知编译器，不需要处理这个属性的解构，能提高性能。

```
class Pair<K, V>(val first: K, val second: V) {  // 1
    operator fun component1(): K {              
        return first
    }

    operator fun component2(): V {              
        return second
    }
}

fun main() {
    val (num, name) = Pair(1, "one")             // 2

    println("num = $num, name = $name")
}
```

- 1 定义一个自己定义的Pair 类，包括自己定义 component1 和 component2 方法。
- 2 解构一个类，就是利用这个pair 这样的方式来做到的。

### Smart Casts 灵活的类型转化

kotlin的*编译器*(这说明可以在ide中就这么操作不需要编码进行类型转化)可以灵活的进行大多数情况的类型转化，包括但不限于以下两种：

1. 将可空类型灵活的转变为对应的不可控类型
2. 从父类型转化为子类型

```
import java.time.LocalDate
import java.time.chrono.ChronoLocalDate

fun main() {
    val date: ChronoLocalDate? = LocalDate.now()    // 1

    if (date != null) {
        println(date.isLeapYear)                    // 2
    }

    if (date != null && date.isLeapYear) {          // 3
        println("It's a leap year!")
    }

    if (date == null || date.isLeapYear) {         // 4
        println("There's no Feb 29 this year...")
    }

    if (date is LocalDate) {
        val month = date.monthValue                 // 5
        println(month)
    }
}
/*
true
It's a leap year!
There's no Feb 29 this year...
10
*/
```

- 1 定义一个可空类型的变量
- 2 判空之后date 自动转变成了他相对应的非空类型
- 3 在这里将会进行一个如2里面的 类型转化，这是由于和java中一样，有判断捷径存在。（即当两个boolean && 的时候第一个为true 才会进行下一个判断，如果第一个为false，将不会进行下一个判断）
- 4 同3 理
- 5 这里便捷地将父类转化成了子类。这一点在java中可以很麻烦地哦

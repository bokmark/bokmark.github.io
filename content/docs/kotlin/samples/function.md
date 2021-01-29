---
title: '【译】Kotlin 语法基础 - 函数'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: '高阶函数、lambda、扩展函数、扩展属性'
---

### 高阶函数

[高阶函数](https://kotlinlang.org/docs/reference/lambdas.html) 是以另一个函数为参数或者返回值的函数。

##### 以另一个函数为参数

```
fun calculate(x: Int, y: Int, operation: (Int, Int) -> Int): Int {  // 1
    return operation(x, y)                                          // 2
}

fun sum(x: Int, y: Int) = x + y                                     // 3

fun main() {
    val sumResult = calculate(4, 5, ::sum)                          // 4
    val mulResult = calculate(4, 5) { a, b -> a * b }               // 5
    println("sumResult $sumResult, mulResult $mulResult")
}
```

- 1 定义一个高阶函数，他有两个int参数 x，y，同时他还有一个函数作为参数：operation。在operation的申明中，他的参数和返回值已经申明。
- 2 这个高阶函数的内部实现是获得operation的返回值在传入x，y的情况下。
- 3 定义一个函数，与operation的定义很像。
- 4 执行这个高阶函数，传入两个int值的参数，和传入一个函数参数`::sum`。`::` 在kotlin中是一个使用名字来提取方法的符号。
- 5 执行这个高阶函数，插入两个int值的参数，和传入一个lambda为参数。这种写法是不是看起来很简洁？

##### 以一个函数作为返回值

```
fun operation(): (Int) -> Int {                                     // 1
    return ::square
}

fun square(x: Int) = x * x                                          // 2

fun main() {
    val func = operation()                                          // 3
    println(func(2))                                                // 4
}
```

- 1 定义一个高阶函数，他返回一个函数作为返回值。在这里`(Int) -> Int`的写法是申明的返回值的函数的结构。这里定义了返回`square`函数
- 2 定义名字叫`square`的函数，他的作用是返回`x*x`
- 3 执行这个高阶函数，获取一个函数作为返回值。在这里，`operation()`返回的`func` 就是`square`方法
- 4 执行`func`。这里`square`方法会被执行、

### lambda 表达式

[Lambda表达式](https://kotlinlang.org/docs/reference/lambdas.html)是一种为了方便创建函数对等式（就是 相当于一种函数）。lambda 可以很简洁地表明函数，通过使用默认参数it

```
// All examples create a function object that performs upper-casing.
// So it's a function from String to String

val upperCase1: (String) -> String = { str: String -> str.toUpperCase() } // 1

val upperCase2: (String) -> String = { str -> str.toUpperCase() }         // 2

val upperCase3 = { str: String -> str.toUpperCase() }                     // 3

// val upperCase4 = { str -> str.toUpperCase() }                          // 4

val upperCase5: (String) -> String = { it.toUpperCase() }                 // 5

val upperCase6: (String) -> String = String::toUpperCase                  // 6

println(upperCase1("hello"))
println(upperCase2("hello"))
println(upperCase3("hello"))
println(upperCase5("hello"))
println(upperCase6("hello"))
```

- 1 A lambda in all its glory, with explicit types everywhere。花括号里面的内容就是lambda，这部分被赋值为类型为 `(String) -> String `的一个内容。
- 2 lambda 内部有发生类型推论。有这个推论的原因是，在外部定义了明确的类型。
- 3 lambda 外部发生了类型推论，这是因为lambda 内部的参数类型和返回值是可推论的
- 4 你不能在lambda 内部和外部同时省略类型，这将无法进行类型推论。
- 5 如果只有一个参数，你可以不定义lambda 内部的参数名字，默认使用it
- 6 如果你的lambda只有一个单一的函数调用，那么你就可以直接使用函数指针`(::)`

### Extension Functions 扩展函数 和 Properties 属性

kotlin 可以通过 [扩展机制](https://kotlinlang.org/docs/reference/extensions.html) 让你扩展任何的class 。顾名思义，有两种扩展方式：扩展方法和扩展属性。他们看起来像是普通的方法和属性，除了一点，你必须明确的说明你是要扩展那个类。

```
data class Item(val name: String, val price: Float)                                   // 1  

data class Order(val items: Collection<Item>)  

fun Order.maxPricedItemValue(): Float = this.items.maxBy { it.price }?.price ?: 0F    // 2  
fun Order.maxPricedItemName() = this.items.maxBy { it.price }?.name ?: "NO_PRODUCTS"

val Order.commaDelimitedItemNames: String                                             // 3
    get() = items.map { it.name }.joinToString()

fun main() {

    val order = Order(listOf(Item("Bread", 25.0F), Item("Wine", 29.0F), Item("Water", 12.0F)))

    println("Max priced item name: ${order.maxPricedItemName()}")                     // 4
    println("Max priced item value: ${order.maxPricedItemValue()}")
    println("Items: ${order.commaDelimitedItemNames}")                                // 5

}
/*
Max priced item name: Wine
Max priced item value: 29.0
Items: Bread, Wine, Water
*/
```

- 1 定义了两个 data class，item  和 order ，其中 order 包含了一个items的属性
- 2 添加一个Order的扩展函数
- 3 添加一个Order的扩展属性
- 4 执行 order的 扩张方法 maxPricedItemName()
- 5 访问 order的扩展属性 commaDelimitedItemNames

甚至你可以为null 扩展方法。在扩展方法中，你可以对this 进行检查是否为null，并根据检查结果做出你的逻辑

```
fun <T> T?.nullSafeToString() = this?.toString() ?: "NULL"  // 1
fun main() {
    println(null.nullSafeToString())
    println("Kotlin".nullSafeToString())
}
```

- 1 定义了一个泛型方法，这个方法可以由null 来调用

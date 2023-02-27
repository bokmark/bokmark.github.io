---
author: Bokmark Ma
title: Kotlin 语法基础「翻译自官网」
date: 2019-03-10
description: Kotlin 语法基础
slug: k/base
categories:
    - Kotlin
tags:
    - kotlin
---

# Kotlin 语法基础

## 【译】Kotlin - 基础介绍

### hello world

```kotlin
package org.kotlinlang.play         // 1

fun main() {                        // 2
    println("Hello, World!")        // 3
}
```

- 1 kotlin 一般定义在包里。但是如果你不定义那么就将内容放到默认的包中。
- 2 `main`方法是程序的入口。 在 Kotlin 1.3 `main`方法可以不定义任何参数也不需要定义返回值。
- 3 `println` 方法将会把一行内容输入到标准输出。这个方法已经默认引用了，不需要显式的引用。分号也不再是必须的。

----

在早于1.3版本`main` 方法必须要有参数

```kotlin
fun main(args: Array<String>) {
    println("Hello, World!")
}
```

----

### 方法

##### 普通方法

```kotlin
fun printMessage(message: String): Unit {                               // 1
    println(message)
}

fun printMessageWithPrefix(message: String, prefix: String = "Info") {  // 2
    println("[$prefix] $message")
}

fun sum(x: Int, y: Int): Int {                                          // 3
    return x + y
}

fun multiply(x: Int, y: Int) = x * y                                    // 4

fun main() {
    printMessage("Hello")                                               // 5                    
    printMessageWithPrefix("Hello", "Log")                              // 6
    printMessageWithPrefix("Hello")                                     // 7
    printMessageWithPrefix(prefix = "Log", message = "Hello")           // 8
    println(sum(1, 2))                                                  // 9
}
```

- 1 一个简单的方法，带有一个string参数和返回unit 就是返回void 没有返回值
- 2 一个带有两个参数的方法，其中 info是一个带有默认值的方法，无返回值就是默认Unit

> 通过tools  show kotlin bytecode 再decompile 可以看到kotlin的java代码长什么样子
>
> ```kotlin
> fun printMessageWithPrefix(message: String, prefix: String = "Info") {  // 2
>  println("[$prefix] $message")
> }
>
> fun main() {
>  printMessageWithPrefix("1")
>  printMessageWithPrefix("1", "2")
> }
> ```
>
> kotlin代码转化之后成为的样子，·从这可以看到kotlin在这里的处理是创建一个不同名字的方法，直接改了调用的地方，不是如预想中的重载·
>
> ```kotlin
> public static final void printMessageWithPrefix(@NotNull String message, @NotNull String prefix) {
>   Intrinsics.checkParameterIsNotNull(message, "message");
>   Intrinsics.checkParameterIsNotNull(prefix, "prefix");
>   String var2 = '[' + prefix + "] " + message;
>   boolean var3 = false;
>   System.out.println(var2);
> }
>
> // $FF: synthetic method
> public static void printMessageWithPrefix$default(String var0, String var1, int var2, Object var3) {
>   if ((var2 & 2) != 0) {
>      var1 = "Info";
>   }
>
>   printMessageWithPrefix(var0, var1);
> }
>
> public static final void main() {
>   printMessageWithPrefix$default("1", (String)null, 2, (Object)null);
>   printMessageWithPrefix("1", "2");
> }
>
> // $FF: synthetic method
> public static void main(String[] var0) {
>   main();
> }
> ```

- 3 一个返回int的值的方法
- 4 只有一个表达式的方法，并且推断返回值为int
- 5 调用了第一个方法传入了一个`hello` 的参数
- 6 调用了第二个方法传入了两个参数
- 7 调用了第二个方法传入了一个参数，第二个参数info带有默认值info
- 8 调用了第二个方法传入了带命名的参数，这使得参数不再是按照参数顺序来传入的
- 9 打印方法的返回值

#####  infix functions 中缀方法

> 如果成员方法和扩展方法只有一个参数那么可以转化为中缀方法

```kotlin
fun main() {

  infix fun Int.times(str: String) = str.repeat(this)        // 1
  println(2 times "Bye ")                                    // 2

  val pair = "Ferrari" to "Katrina"                          // 3
  println(pair)

  infix fun String.onto(other: String) = Pair(this, other)   // 4
  val myPair = "McLaren" onto "Lucas"
  println(myPair)

  val sophia = Person("Sophia")
  val claudia = Person("Claudia")
  sophia likes claudia                                       // 5
}

class Person(val name: String) {
  val likedPeople = mutableListOf<Person>()
  infix fun likes(other: Person) { likedPeople.add(other) }  // 6
}
```

- 1 定义一个中缀扩展方法在 Int 类里
- 2 嗲用这个方法
- 3 创建一个pair 通过标准库里的中缀方法 `to`
- 4 创建一个你自己的 类似于`to`的方法 `onto`
- 5 中缀符号 也可以定义成 成员方法，用于类对象使用
- 6 定义了中缀方法的类将会成为这个方法里面的第一个参数，即有个默认的this

> local functions 内部方法
> 定义在方法中的方法

##### 操作符方法

明确的方法可以升级为 操作符方法，允许调用者使用相应的符号进行运算

```kotlin
operator fun Int.times(str: String) = str.repeat(this)       // 1
println(2 * "Bye ")                                          // 2

operator fun String.get(range: IntRange) = substring(range)  // 3
val str = "Always forgive your enemies; nothing annoys them so much."
println(str[0..14])                                          //14
```

- 1 操作符和对应的重载方法是一一对应的 times 方法就是对应 *的,相当于重载操作符了
- 2 `times` 对应了 `*` 你调用 `2 * "Bye "` 相当与 调用了 `2.times("Bye ")`
- 3 get(range: IntRange) ==> [0 .. 14]
- 4 [可以参考这篇文章](https://kotlinlang.org/docs/reference/operator-overloading.html#indexed)

##### 带有 vararg 参数的方法

> vararg 就相当于 java 中 String... args

```kotlin
fun printAll(vararg messages: String) {                            // 1
    for (m in messages) println(m)
}
printAll("Hello", "Hallo", "Salut", "Hola", "你好")                 // 2

fun printAllWithPrefix(vararg messages: String, prefix: String) {  // 3
    for (m in messages) println(prefix + m)
}
printAllWithPrefix(
    "Hello", "Hallo", "Salut", "Hola", "你好",
    prefix = "Greeting: "                                          // 4
)

fun log(vararg entries: String) {
    printAll(*entries)                                             // 5
}
```

-  1 如何获取 vararg中的每个参数
-  2 你也可以传入任意数量的参数包括0个
-  3 由于有命名参数的存在你可以在可变长度的参数后面再添加一个同类型的参数，二这一点在java中是做不到的
-  4 你应该像这样一样使用
-  5 在运行时中，vararg 是以数组的形式传递到参数中，这和java是一样的。你可以用`*` 将可变长度的参数以可变长度的参数来传递而不是以数组的方式来传递。这里不加 * 将会报错。

---

### 变量

kotlin 具有强大的变量的推断能力。你可以明确的定义变量的类型，也可以让编译器自己来推断变量类型是什么。推荐使用val 来定义变量，因为这是不变的变量，当然kotlin 也不会强制你将变量定义为不变，你可以使用var。

```kotlin
var a: String = "initial"  // 1
println(a)
val b: Int = 1             // 2
val c = 3                  // 3
```

- 1 定义了一个可变变量，并且初始化
- 2 定义了一个不可变变量，并且初始化。相当于java中添加了 final
- 3 定义了一个不可变变量，并且初始化，并且没有明确定义变量名称。类型将由编译器推断，这里编译器推断的是Int

```kotlin
var e: Int  // 1
println(e)  // 2
```

- 1 定义了一个变量并且没有对他进行初始化
- 2 这里将会报错，变量必须进行初始化

> 你可以在任何地方进行初始化，只要在read 这个变量之前就可以。这里和java 不同

```kotlin
val d: Int  // 1

if (someCondition()) {
    d = 1   // 2
} else {
    d = 2   // 2
}

println(d) // 3
```

- 1 定义一个不可变的变量
- 2 根据一些条件进行判断 之后再初始化
- 3 read 这个变量，这里不报错，因为已经初始化了

---

### 空安全检查

kotlin 想要创造一个没有`NullPointerException`的世界，所以不允许给一个变量赋予Null值。如果你确实需要赋予一个变量null，请在他的类型名称之后加上`?`

```kotlin
var neverNull: String = "This can't be null"            // 1

neverNull = null                                        // 2

var nullable: String? = "You can keep a null here"      // 3

nullable = null                                         // 4

var inferredNonNull = "The compiler assumes non-null"   // 5

inferredNonNull = null                                  // 6

fun strLength(notNull: String): Int {                   // 7
    return notNull.length
}

strLength(neverNull)                                    // 8
strLength(nullable)                                     // 9
```

- 1 定义了一个不可为空的String，并且赋值
- 2 将这个字符串设置为null（这里将会报错）
- 3 定义一个可为空的String，并且赋值
- 4 将这个字符串设置为null，这里没有报错
- 5 定义一个由编译器进行推断的变量，这里编译器推断为String。编译器将会把变量优先推断为非空类型
- 6 报错，因为编译器进行推断的时候都将
- 7 定义了一个方法，传入的是非空的参数String
- 8 传入的是非空的String值，这里将不会报错
- 9 传入的可为空的String? 值，这里将会报错
  有时候你必须处理null值，可能是因为Java代码传入了null 也可能是null代表了一种状态，这时候kotlin提供了一种处理null的方法

```kotlin
fun describeString(maybeString: String?): String {              // 1
    if (maybeString != null && maybeString.length > 0) {        // 2
        return "String of length ${maybeString.length}"
    } else {
        return "Empty or null string"                           // 3
    }
}
```

- 1 定义了一个方法可入可为空的String 类型，返回一个不可为空的String类型
- 2 判断maybeString 是否为空或者为空字符串，如果是就返回maybeString的长度
- 3 不然的话就告知调用者，传入的参数为空或者为空字符串

---

### 类

类[定义](https://kotlinlang.org/docs/reference/classes.html?&_ga=2.241391169.2133596250.1603679551-1735323035.1603280899#classes)由类名，类头（包含类型声明，私有构造方法等）、和类体组成，并且由大括号组成。类头和类体都是可以缺省的。如果类体缺省，那么大括号也可以缺省。

```kotlin
class Customer                                  // 1

class Contact(val id: Int, var email: String)   // 2

fun main() {

    val customer = Customer()                   // 3

    val contact = Contact(1, "mary@gmail.com")  // 4

    println(contact.id)                         // 5
    contact.email = "jane@gmail.com"            // 6
}
```

- 1 定义一个类Customer ，这个类没有定义任何属性和构造方法。 kotlin将会自动赋予一个无参构造方法
- 2 定义一个Contact 类，类里面有一个不可变变量的id，一个可变的变量email，并且有一个带有两个参数的构造方法
- 3 创建一个不可变的变量 实例customer类型推断为Customer。使用Customer的默认构造方法。在kotlin中是没有new关键字的
- 4 创建一个Contact的实例，使用了带有两个参数的构造方法
- 5 读取contact的属性id
- 6 更新contact的属性email

---

### 泛型

泛型是现代语言都具有的一种模板方法。泛型类和泛型方法使得代码具有高度的可复用性。就像`List<T>`里卖弄的内部逻辑与T就没有什么逻辑上的关系

##### 泛型类

第一种使用泛型的方法就是泛型类

```kotlin
class MutableStack<E>(vararg items: E) {              // 1

  private val elements = items.toMutableList()

  fun push(element: E) = elements.add(element)        // 2

  fun peek(): E = elements.last()                     // 3

  fun pop(): E = elements.removeAt(elements.size - 1)

  fun isEmpty() = elements.isEmpty()

  fun size() = elements.size

  override fun toString() = "MutableStack(${elements.joinToString()})"
}
```

- 1 定义一个泛型类，E可以被称为泛型参数。当你定义`MutableStack<Int>` 的时候Int 就被称呼为泛型参数
- 2 在泛型类里面的方法中你可以用E来代替泛型参数
- 3 你也可以用E来代替返回值

##### 泛型方法

你当然也可以定义[泛型方法](https://kotlinlang.org/docs/reference/generics.html?&_ga=2.253449418.2133596250.1603679551-1735323035.1603280899#generic-functions)，如果他们的逻辑与泛型参数无关的话。比如你可以定义一个这样的方法

```kotlin
fun <E> mutableStackOf(vararg elements: E) = MutableStack(*elements)

fun main() {
  val stack = mutableStackOf(0.62, 3.14, 2.7)
  println(stack)
}
```

 注意：编译将会推断出mutableStackOf是一个Double的MutableStack所以你不需要填写`mutableStackOf<Double>(...)` 之类的

---

### 继承

kotlin 支持传统的面向对象的继承机制

```kotlin
open class Dog {                // 1
    open fun sayHello() {       // 2
        println("wow wow!")
    }
}

class Yorkshire : Dog() {       // 3
    override fun sayHello() {   // 4
        println("wif wif!")
    }
}

fun main() {
    val dog: Dog = Yorkshire()
    dog.sayHello()
}
```

- 1 kotlin 的类默认是final的，如果你要允许类可以被继承，你需要添加 open 关键字到class 之前。
- 2 kotlin的方法默认也是final的，如果你要允许方法可以被重写，你需要添加open关键字到fun 之前。
- 3 一个子类应该定义在` : SuperclassName()`之前。空的`()` 意味着super了父类的默认构造方法。
- 4 重写的方法胡总和属性，应该添加override 关键字

##### 带有参数的构造方法的继承

```kotlin
open class Tiger(val origin: String) {
    fun sayHello() {
        println("A tiger from $origin says: grrhhh!")
    }
}

class SiberianTiger : Tiger("Siberia")                  // 1

fun main() {
    val tiger: Tiger = SiberianTiger()
    tiger.sayHello()
}
```

- 1 如果你想要用父类的构造方法定义一个子类，那么你需要在定义的时候传入传给父类的字段。

##### 传递参数给父类

```kotlin
open class Lion(val name: String, val origin: String) {
    fun sayHello() {
        println("$name, the lion from $origin says: graoh!")
    }
}

class Asiatic(name: String) : Lion(name = name, origin = "India") // 1

fun main() {
    val lion: Lion = Asiatic("Rufo")                              // 2
    lion.sayHello()
}
```

- 1 `Asiatic` 类里的name 既不是val 也不是var。这是一个构造方法的参数，只用来将值传递给父类的的构造方法。
- 2 用构造参数Rufo创建一个Asiatic的实例，这个调用将会嗲用父类`Lion` 的双参构造方法




## 【译】Kotlin 语法基础 - 函数


### 高阶函数

[高阶函数](https://kotlinlang.org/docs/reference/lambdas.html) 是以另一个函数为参数或者返回值的函数。

##### 以另一个函数为参数

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
fun <T> T?.nullSafeToString() = this?.toString() ?: "NULL"  // 1
fun main() {
    println(null.nullSafeToString())
    println("Kotlin".nullSafeToString())
}
```

- 1 定义了一个泛型方法，这个方法可以由null 来调用



## 【译】Kotlin 语法基础 - 作用域方法


### let

let 是一个kotlin 的标准库，可以用于实例检查和null检查，当let被调用再一个实例上的时候，你可以通过it 来再作用域内访问这个实例。作用域内的最后一句表达式，将会作为返回值返回

```kotlin
fun customPrint(s: String) {
    print(s.toUpperCase())
}

fun main() {
    val empty = "test".let {               // 1
        customPrint(it)                    // 2
        it.isEmpty()                       // 3
    }
    println(" is empty: $empty")


    fun printNonNull(str: String?) {
        println("Printing \"$str\":")

        str?.let {                         // 4
            print("\t")
            customPrint(it)
            println()
        }
    }
    printNonNull(null)
    printNonNull("my string")
}
/*
TEST is empty: false
Printing "null":
Printing "my string":
    MY STRING
*/
```

- 1 在“test”上执行这一个代码块，并且返回一个值作为结果
- 2 调用一个方法，通过it访问到“test”实例
- 3 返回let代码块的返回值
- 4 安全调用let代码块，只有当str不为空是才会执行let代码块。

___

### run

run 和let 很相似，执行一个代码块，返回一个结果。两者不同的是 let中使用it 来访问外部调用实例，而run中使用this 来访问外部调用实例，这在嗲用外部调用实例的方法是会提供一些便捷。

```kotlin
fun main() {
    fun getNullableLength(ns: String?) {
        println("for \"$ns\":")
        ns?.run {                                                  // 1
            println("\tis empty? " + isEmpty())                    // 2
            println("\tlength = $length")                           
            length                                                 // 3
        }
    }
    getNullableLength(null)
    getNullableLength("")
    getNullableLength("some string with Kotlin")
}
/*

for "null":
for "":
    is empty? true
    length = 0
for "some string with Kotlin":
    is empty? false
    length = 23
*/
```

- 1 在一个可控变量上执行一个代码块
- 2 不需要显示的调用this.xxx 直接调用 isempty， 这里就相当于调用 this.isEmpty()
- 3 返回this.length

___

### with

```kotlin
class Configuration(var host: String, var port: Int)

fun main() {
    val configuration = Configuration(host = "127.0.0.1", port = 9000)

    with(configuration) {
        println("$host:$port")
    }

    // instead of:
    println("${configuration.host}:${configuration.port}")    
}
/*
127.0.0.1:9000
127.0.0.1:9000
*/
```

 with是一个非扩展方法，在with的作用域里，你可以简洁地访问到参数地成员，意思就是在作用域里面你可以隐藏实例地名字当访问属性地时候。

___

### apply

apply 在一个实例上运行一段代码，让后将返回这个实例。在代码块中，实例可以用this来代替访问。这个apply，可以简单地用于初始化实例。

```kotlin
data class Person(var name: String, var age: Int, var about: String) {
    constructor() : this("", 0, "")
}

fun main() {
    val jake = Person()                                     // 1
    val stringDescription = jake.apply {                    // 2
        name = "Jake"                                       // 3
        age = 30
        about = "Android developer"
    }.toString()                                            // 4
    println(stringDescription)
}
//Person(name=Jake, age=30, about=Android developer)
```

- 1 创建一个实例
- 2 调用一个apply 代码块
- 3 修改属性
- 4 apply返回地还是 这个实例，在这里可以调用 toString(), 这样你就可以链式地调用。

___

### also

also 和apply的工作方式很像，他执行了一个代码块，然后返回这个实例。在代码块中，你需要使用it来访问这个实例，这是和apply不同的地方。所以这更像是用来执行一个额外的工作。比如在调用链中打印一些东西

```kotlin
data class Person(var name: String, var age: Int, var about: String) {
             constructor() : this("", 0, "")
}

fun writeCreationLog(p: Person) {
    println("A new person ${p.name} was created.")              
}

fun main() {
    val jake = Person("Jake", 30, "Android developer")   // 1
        .also {                                          // 2
            writeCreationLog(it)                         // 3
        }
}
//A new person Jake was created.
```

- 1 创建一个带有属性值的 Person
- 2 调用also代码块，代码块的返回值就是他自己
- 3 调用一个方法，打印一些内容


## 【译】Kotlin 语法基础 - 流程控制



### when

##### when 语句

用来代替广泛使用的 switch 关键字，Kotlin 提供了一种更加灵活和简洁的when 结构。when可以被用来当作一个语句，也可以用来当作一个表达式

```kotlin
fun main() {
    cases("Hello")
    cases(1)
    cases(0L)
    cases(MyClass())
    cases("hello")
}

fun cases(obj: Any) {                                
    when (obj) {                                     // 1   
        1 -> println("One")                          // 2
        "Hello" -> println("Greeting")               // 3
        is Long -> println("Long")                   // 4
        !is String -> println("Not a string")        // 5
        else -> println("Unknown")                   // 6
    }   
}

class MyClass

/*
Greeting
One
Long
Not a string
Unknown
*/
```

- 1 这是一个`when`语句
- 2 检查哪个obj 等于1
- 3 检查哪个obj 等于 Hello
- 4 检查哪个obj 是Long
- 5 检查哪个obj 不是String
- 6 默认语句
  注意：将会按照顺序一条一条语句查询，直到一条语句满足条件，所以只有最靠前的满足条件的语句才会被执行

##### when 表达式

```kotlin
fun main() {
    println(whenAssign("Hello"))
    println(whenAssign(3.4))
    println(whenAssign(1))
    println(whenAssign(MyClass()))
}

fun whenAssign(obj: Any): Any {
    val result = when (obj) {                   // 1
        1 -> "one"                              // 2
        "Hello" -> 1                            // 3
        is Long -> false                        // 4
        else -> 42                              // 5
    }
    return result
}

class MyClass
```

- 1 这是一个when 语句表达式
- 2 检查哪个obj等于1 如果等于返回一个one的字符串
- 3 检查哪个obj等于 Hello 如果等于返回一个 1
- 4 检查哪个obj 是Long 类型，如果等于返回一个false
- 5 默认返回 42。*不像在 when语句中，在when表达式中else 式要求一定存在的，因为如果不存在，编译器将无法覆盖所有的可能性来推断出 返回值的类型*

---

### Loop 循环

kotlin 支持的循环模式有以下几种：`for` , `while` , `do-while`

##### for

`for` 在kotlin的使用方式与大多数语言一致

```kotlin
val cakes = listOf("carrot", "cheese", "chocolate")

for (cake in cakes) {                               // 1
    println("Yummy, it's a $cake cake!")
}
```

- 1 将会遍历每个参数

##### while 和 do-while

`while` 和 `do-while` 在kotlin的使用方式与大多数语言一致

```kotlin
fun eatACake() = println("Eat a Cake")
fun bakeACake() = println("Bake a Cake")

fun main(args: Array<String>) {
    var cakesEaten = 0
    var cakesBaked = 0

    while (cakesEaten < 5) {                    // 1
        eatACake()
        cakesEaten ++
    }

    do {                                        // 2
        bakeACake()
        cakesBaked++
    } while (cakesBaked < cakesEaten)

}
```

- 1 执行语句块当判断条件为true的时候
- 2 先执行语句块再判断条件

##### Iterators 迭代器

你可以再类中自定义你的迭代器，通过再类里 实现 iterator 操作符 方法

```kotlin
class Animal(val name: String)

class Zoo(val animals: List<Animal>) {

    operator fun iterator(): Iterator<Animal> {             // 1
        return animals.iterator()                           // 2
    }
/*                                                          // 2.1
  operator fun iterator(): Iterator<Animal> {          
        return object :   Iterator<Animal> {
            override fun next() : Animal {
                return Animal("111")
            }
            override fun hasNext(): Boolean {
                return false
            }
        }                       
    }
*/
}

fun main() {

    val zoo = Zoo(listOf(Animal("zebra"), Animal("lion")))

    for (animal in zoo) {                                   // 3
        println("Watch out, it's a ${animal.name}")
    }

}
/*
Watch out, it's a zebra
Watch out, it's a lion
*/
```

- 1 在类中定义一个迭代器。这个操作符方法的方法名为 iterator，用operator 修复fun
- 2 返回一个迭代器这意味着你必须实现这两个方法 例如2.1那样
  `next(): Animal`
  `hasNext(): Boolean`
- 3 通过使用用户定义的迭代器进行循环

The iterator can be declared in the type or as an extension function.

### Ranges 区间

kotlin提供了一系列工具用于区间的数据处理。我们大致看一下

```kotlin
for(i in 0..3) {             // 1
    print(i)
}
print(" ")

for(i in 2..8 step 2) {      // 2
    print(i)
}
print(" ")

for (i in 3 downTo 0) {      // 3
    print(i)
}
print(" ")
// 0123 2468 3210
```

- 1 从0到3 一个一个遍历，包含0 和3 。输出0 1 2 3
- 2 从2到8 两个两个遍历，包含2和8。输出 2 4 6 8
- 3 从3到0倒序一个一个遍历，包含3和0。输出 3 2 1 0

char数组也支持同样的操作

```kotlin
for (c in 'a'..'d') {        // 1
    print(c)
}
print(" ")

for (c in 'z' downTo 's' step 2) { // 2
    print(c)
}
print(" ")
// abcd zxvt
```

-   1 按照字母表遍历 a 到 d 包括a和 d
-   2 按照字母表从z 倒序到 s 两个两个遍历。char 范围同样支持 step 和downTo 关键字

in关键字 同样支持用在if语句中

```kotlin
val x = 2
if (x in 1..5) {            // 1
    print("x is in range from 1 to 5")
}
println()

if (x !in 6..10) {          // 2
    print("x is not in range from 6 to 10")
}
```

- 1 判断一个x 是否在1 - 5的范围里
- 2 判断x 是否不在 6 - 10 的范围中

----

### 相等关系检查 Equality Checks

在kotlin 中，使用 `==` 来比较结构是否相等。使用` ===` 来比较引用地址是否相等。
更准确地说，`a == b` 编译完之后等价于 ` if (a == null) b == null else a.equals(b)`.

```kotlin
val authors = setOf("Shakespeare", "Hemingway", "Twain")
val writers = setOf("Twain", "Shakespeare", "Hemingway")
val writers1 = setOf("Twain", "Shakespeare", "Hemingway")
val writers2 = writers

println(authors == writers)    // 1
println(authors === writers)   // 2
println(writers === writers1)  // 3
println(writers2 === writers)  // 4
/*
true
false
false
true
*/
```

- 1 返回 `true` 因为相当于调用了 `authors.equals(writers)` 然后sets将会忽略元素的顺序
- 2 返回 `false` 因为 `authors` 和 `writers`不是同样的引用地址.
- 3 返回 `false` 哪怕writers 和writers1 完全相同，两者指向的也不是相同的引用地址
- 4 返回 `true` 指向了同一个对象

##### 多元表达式 Conditional Expression

在kotlin中没有java 的哪种 `a?b:c` 的三元表达式。取而代之地是使用if表达式

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b         // 1

println(max(99, -42))
```

- 1  返回 a或者b

## 【译】Kotlin 语法基础 - 集合


### [List](https://kotlinlang.org/docs/reference/collections-overview.html)
list 是一个有序的集合。在kotlin中 list 可以是可变的 `MutableList ` 也可以是只读的 `List `。在创建list的时候，使用标准类库里的 `listOf()` 创建只读的不可变的 `List`，使用 `mutableListOf()` 创建可变的list。为了避免不必要的修改，你可以用List的引用来指向可变list， 这将避免错误的修改。

```kotlin
val systemUsers: MutableList<Int> = mutableListOf(1, 2, 3)        // 1
val sudoers: List<Int> = systemUsers                              // 2

fun addSudoer(newUser: Int) {                                     // 3
    systemUsers.add(newUser)                      
}

fun getSysSudoers(): List<Int> {                                  // 4
    return sudoers
}

fun main() {
    addSudoer(4)                                                  // 5
    println("Tot sudoers: ${getSysSudoers().size}")               // 6
    getSysSudoers().forEach {                                     // 7
        i -> println("Some useful info on user $i")
    }
    // getSysSudoers().add(5) <- Error!                           // 8
}
```
- 1 创建一个可变的list
- 2 通过一个`List `的对象来指向 可变list。这将避免对`List `进行修改
- 3 定义一个方法，将一个int 值添加到 可变list
- 4 定义一个方法，来获取可变list的一个 不可变引用。
- 5 调用`addSudoer` 方法
- 6 获取不可变list 引用 然后读取size
- 7 获取不可变list 引用再循环遍历
- 8 获取不可变list 然后调用add 方法将会导致报错

___
### Set
set 是一种无序的集合，并且不存在重复的健值。创建set，可以使用`setOf() `和 `mutableSetOf()`。和list 一样，set 可以通过一个不可变的引用来避免不必要的修改。

```kotlin
val openIssues: MutableSet<String> = mutableSetOf("uniqueDescr1", "uniqueDescr2", "uniqueDescr3") // 1

fun addIssue(uniqueDesc: String): Boolean {                                                       
    return openIssues.add(uniqueDesc)                                                             // 2
}

fun getStatusLog(isAdded: Boolean): String {                                                       
    return if (isAdded) "registered correctly." else "marked as duplicate and rejected."          // 3
}

fun main() {
    val aNewIssue: String = "uniqueDescr4"
    val anIssueAlreadyIn: String = "uniqueDescr2"

    println("Issue $aNewIssue ${getStatusLog(addIssue(aNewIssue))}")                              // 4
    println("Issue $anIssueAlreadyIn ${getStatusLog(addIssue(anIssueAlreadyIn))}")                // 5
}
```
- 1 创建一个可变的set
- 2 定义一个方法往可变set里面添加元素，set的add 将会返回添加是否成功的结果
- 3 定义一个方法
- 4 打印添加成功的结果
- 5 打印添加失败的结果
____

### Map
map 是一个键值对的集合，键是唯一的，并且是用来找到相关的value值的。创建 map 同样有两种方法`mapOf` 和 `mutableMapOf `。使用` to `的中缀方法可以快速的创建map 。和list 一样，map 可以通过一个不可变的引用来避免不必要的修改。

```kotlin
const val POINTS_X_PASS: Int = 15
val EZPassAccounts: MutableMap<Int, Int> = mutableMapOf(1 to 100, 2 to 100, 3 to 100)   // 1
val EZPassReport: Map<Int, Int> = EZPassAccounts                                        // 2

fun updatePointsCredit(accountId: Int) {
    if (EZPassAccounts.containsKey(accountId)) {                                        // 3
        println("Updating $accountId...")                                               
        EZPassAccounts[accountId] = EZPassAccounts.getValue(accountId) + POINTS_X_PASS  // 4
    } else {
        println("Error: Trying to update a non-existing account (id: $accountId)")
    }
}

fun accountsReport() {
    println("EZ-Pass report:")
    EZPassReport.forEach {                                                              // 5
        k, v -> println("ID $k: credit $v")
    }
}

fun main() {
    accountsReport()                                                                    // 6
    updatePointsCredit(1)                                                               // 7
    updatePointsCredit(1)                                                               
    updatePointsCredit(5)                                                               // 8
    accountsReport()                                                                    // 9
}
```
- 1 创建一个可变的map ，并且展示了 to的用法
- 2 创建一个不可变的引用
- 3 检查key是否已经存在
- 4 读取key相对应的value ，并且将其修改
- 5 遍历不可变map 和输出他的键值对
- 6 在修改之前，先打印出结果
- 7 修改两次存在的key的对应的value
- 8 修改不存在的key对应的value
- 9 再打印一遍结果
___

### filter
过滤方法可以允许你对集合进行过滤，filter 需要你提供一个lambda的参数，lambda函数将会对每一个元素进行判断，如果判断的是true，那么久将结果加入到返回的集合中去。

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)      // 1

    val positives = numbers.filter { x -> x > 0 }  // 2

    val negatives = numbers.filter { it < 0 }      // 3

    println("Numbers: $numbers")
    println("Positive Numbers: $positives")
    println("Negative Numbers: $negatives")
}
/*
Numbers: [1, -2, 3, -4, 5, -6]
Positive Numbers: [1, 3, 5]
Negative Numbers: [-2, -4, -6]
*/
```
- 1 创建一个list
- 2 过滤保留大于0的元素
- 3 过滤保留小于0的元素
___

### map
map 这个扩展方法，可以让你对集合中的每个元素做一个转化，你需要的是传入一个lambda的函数，作为转化所必须的元素。

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)     // 1

    val doubled = numbers.map { x -> x * 2 }      // 2

    val tripled = numbers.map { it * 3 }          // 3

    println("Numbers: $numbers")
    println("Doubled Numbers: $doubled")
    println("Tripled Numbers: $tripled")
}
/*
Numbers: [1, -2, 3, -4, 5, -6]
Doubled Numbers: [2, -4, 6, -8, 10, -12]
Tripled Numbers: [3, -6, 9, -12, 15, -18]
*/
```
- 1 创建一个list
- 2 所有元素都 * 2
- 3 所有元素都 * 3，这里可以便捷的使用it，因为这里可以进行类型推断
____

### any, all, none
这几个方法用于检查集合中的方法是否符合给出的条件

##### any
any 方法当集合中至少有一个符合给出的条件就返回true

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)            // 1

    val anyNegative = numbers.any { it < 0 }             // 2

    val anyGT6 = numbers.any { it > 6 }                  // 3

    println("Numbers: $numbers")
    println("Is there any number less than 0: $anyNegative")
    println("Is there any number greater than 6: $anyGT6")
/*
Numbers: [1, -2, 3, -4, 5, -6]
Is there any number less than 0: true
Is there any number greater than 6: false
*/
}
```
- 1 定义一个list
- 2 是否有一个元素小于0
- 3 是否有一个元素大于6

##### all
当集合中所有的元素都符合条件的时候返回true

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)            // 1

    val allEven = numbers.all { it % 2 == 0 }            // 2

    val allLess6 = numbers.all { it < 6 }                // 3

    println("Numbers: $numbers")
    println("All numbers are even: $allEven")
    println("All numbers are less than 6: $allLess6")
/*
Numbers: [1, -2, 3, -4, 5, -6]
All numbers are even: false
All numbers are less than 6: true
*/
}
```
- 1 定义一个数组
- 2 判断是否所有元素都是偶数
- 3 判断是否所有元素都小于6

##### none
当没有一个元素是符合给出的条件的是否返回true

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)            // 1

    val allEven = numbers.none { it % 2 == 1 }           // 2

    val allLess6 = numbers.none { it > 6 }               // 3

    println("Numbers: $numbers")
    println("All numbers are even: $allEven")
    println("No element greater than 6: $allLess6")
/*
Numbers: [1, -2, 3, -4, 5, -6]
All numbers are even: false
No element greater than 6: true
*/
}
```
- 1 定义一个list
- 2 检查是否没有元素是奇数
- 3 检查是否没有元素都是大于6
----
### find, findLast
查找集合中符和条件的第一个元素或者最后一个元素，如果查找不到就返回null。

```kotlin
fun main() {

    val words = listOf("Lets", "find", "something", "in", "collection", "somehow")  // 1

    val first = words.find { it.startsWith("some") }                                // 2
    val last = words.findLast { it.startsWith("some") }                             // 3

    val nothing = words.find { it.contains("nothing") }                             // 4

    println("The first word starting with \"some\" is \"$first\"")
    println("The last word starting with \"some\" is \"$last\"")
    println("The first word containing \"nothing\" is ${nothing?.let { "\"$it\"" } ?: "null"}")
}
/*
The first word starting with "some" is "something"
The last word starting with "some" is "somehow"
The first word containing "nothing" is null
*/
```
- 1 创建一个list
- 2 查找以some开头的第一个元素
- 3 察州以some开头的最后一个元素
- 4 查找包含noting的第一个元素

----
### first, last
##### first, last
返回相应的集合的第一个或者最后一个元素。你可以传入lambda表达式作为条件来筛选 第一个或者最后一个元素。这个方法会是一种断言的方法，如果集合为空，或者不包含符和条件的元素将会抛出 `NoSuchElementException`

```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)            // 1

    val first = numbers.first()                          // 2
    val last = numbers.last()                            // 3

    val firstEven = numbers.first { it % 2 == 0 }        // 4
    val lastOdd = numbers.last { it % 2 != 0 }           // 5

    println("Numbers: $numbers")
    println("First $first, last $last, first even $firstEven, last odd $lastOdd")
}
```
- 1 创建一个list
- 2 读取list的第一个元素
- 3 读取list的最后一个元素
- 4 读取第一个符和条件的元素，及第一个偶数
- 5 读取最后一个符和条件的元素，及最后一个奇数

##### firstOrNull, lastOrNull
这些方法与上面两个方法不同在于，如果没有符和条件的元素，将会返回null

```kotlin
fun main() {

    val words = listOf("foo", "bar", "baz", "faz")         // 1
    val empty = emptyList<String>()                        // 2

    val first = empty.firstOrNull()                        // 3
    val last = empty.lastOrNull()                          // 4

    val firstF = words.firstOrNull { it.startsWith('f') }  // 5
    val firstZ = words.firstOrNull { it.startsWith('z') }  // 6
    val lastF = words.lastOrNull { it.endsWith('f') }      // 7
    val lastZ = words.lastOrNull { it.endsWith('z') }      // 8

   println("First $first, last $last")
   println("First starts with 'f' is $firstF, last starts with 'z' is $firstZ")
   println("First ends with 'f' is $lastF, last ends with 'z' is $lastZ")
}
```
- 1 定义一个list
- 2 定义一个空list
- 3 读取空list的第一个元素，这里返回null，不会报错
- 4 读取空list的最后一个元素，这里返回null，不会报错
- 5 查找符和条件的第一个元素，结果是foo
- 6 查找符和条件的第一个元素，结果是null
- 7 查找符和条件的最后一个元素，结果是null
- 8 查找符和条件的最后一个元素，结果是faz
___

### count
count 方法将会返回集合的元素个数，或者返回符和条件的集合的元素个数
```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)            // 1

    val totalCount = numbers.count()                     // 2
    val evenCount = numbers.count { it % 2 == 0 }        // 3

    println("Total number of elements: $totalCount")
    println("Number of even elements: $evenCount")
}
```
- 1 定义一个llist
- 2 计算数量
- 3 计算符和条件的数量，这里是计算所有偶数的数量
---

### associateBy, groupBy
两个方法都用于从一个集合中通过对key 进行归类然后来创建一个map。key 可以通过使用keySelector参数来选择，你也可以通过valueSelector 来进行value的选择。

两者之间的区别
- associateBy：value 会使用符和条件的最后一个元素
- groupBy：会把所有符和条件的元素作为放到一个数组中 作为value
返回的map中的entry的顺序维持了原来的list的顺序。
```kotlin
fun main() {


    data class Person(val name: String, val city: String, val phone: String) // 1

    val people = listOf(                                                     // 2
        Person("John", "Boston", "+1-888-123456"),
        Person("Sarah", "Munich", "+49-777-789123"),
        Person("Svyatoslav", "Saint-Petersburg", "+7-999-456789"),
        Person("Vasilisa", "Saint-Petersburg", "+7-999-123456"))

    val phoneBook = people.associateBy { it.phone }                          // 3
    val cityBook = people.associateBy(Person::phone, Person::city)           // 4
    val peopleCities = people.groupBy(Person::city, Person::name)            // 5


    println("People: $people")
    println("Phone book: $phoneBook")
    println("City book: $cityBook")
    println("People living in each city: $peopleCities")
}
/*
People: [Person(name=John, city=Boston, phone=+1-888-123456), Person(name=Sarah, city=Munich, phone=+49-777-789123), Person(name=Svyatoslav, city=Saint-Petersburg, phone=+7-999-456789), Person(name=Vasilisa, city=Saint-Petersburg, phone=+7-999-123456)]
Phone book: {+1-888-123456=Person(name=John, city=Boston, phone=+1-888-123456), +49-777-789123=Person(name=Sarah, city=Munich, phone=+49-777-789123), +7-999-456789=Person(name=Svyatoslav, city=Saint-Petersburg, phone=+7-999-456789), +7-999-123456=Person(name=Vasilisa, city=Saint-Petersburg, phone=+7-999-123456)}
City book: {+1-888-123456=Boston, +49-777-789123=Munich, +7-999-456789=Saint-Petersburg, +7-999-123456=Saint-Petersburg}
People living in each city: {Boston=[John], Munich=[Sarah], Saint-Petersburg=[Svyatoslav, Vasilisa]}
*/
```
- 1 定义一个data class
- 2 定义一个people的list
- 3 构建一个map，这个map的key是 people的的phone， value没有写，就默认为自身。
- 4 构建一个map，key为people的phone，value 为people的city
- 5 构建一个map，key为city，value为city相同的所有people的name
___

### partition
用于将一个list分为两个部分，
```kotlin
fun main() {

    val numbers = listOf(1, -2, 3, -4, 5, -6)                // 1

    val evenOdd = numbers.partition { it % 2 == 0 }           // 2
    val (positives, negatives) = numbers.partition { it > 0 } // 3

    println("Numbers: $numbers")
    println("Even numbers: ${evenOdd.first}")
    println("Odd numbers: ${evenOdd.second}")
    println("Positive numbers: $positives")
    println("Negative numbers: $negatives")
}
```
___

### flatMap
用于遍历集合中的每个元素，然后将所有的结果放置到一个单独的数组中去。转化方式由用户定义。
```kotlin
fun main() {

    val numbers1 = listOf("1", "2", "3")                        // 1
    val numbers = listOf(1, 2, 3)                        

    val tripled = numbers.flatMap { listOf(it, it, it) }        // 2
    val tripled1 = numbers1.flatMap { listOf(it, it + 1) }
    println("Numbers: $numbers1")
    println("Transformed: $tripled1")

    println("Numbers: $numbers")
    println("Transformed: $tripled")
}
```
- 1 定义一个数字的数组
- 2 关键点在于返回的结果不是一个list的list 而是一个list 有九个元素
___

### min, max
选出集合中最大和最小的元素，其中如果集合为空，则返回null
```kotlin
fun main() {


    val numbers = listOf(1, 2, 3)
    val empty = emptyList<Int>()
    val numbers1 = listOf("1", "2", "3")

    println("Numbers: $numbers, min = ${numbers.min()} max = ${numbers.max()}") // 1
    println("Empty: $empty, min = ${empty.min()}, max = ${empty.max()}")        // 2
    println("Numbers: $numbers1, min = ${numbers1.min()} max = ${numbers1.max()}") // 3
}
/*
Numbers: [1, 2, 3], min = 1 max = 3
Empty: [], min = null, max = null
*/
```
___

### sorted
返回一个集合的list 根据他们的自然排列顺序（升序）
sortedBy 根据某个规则按照升序的方式排列
```kotlin
import kotlin.math.abs

fun main() {

    val shuffled = listOf(5, 4, 2, 1, 3, -10)                   // 1
    val natural = shuffled.sorted()                             // 2
    val inverted = shuffled.sortedBy { -it }                    // 3
    val descending = shuffled.sortedDescending()                // 4
    val descendingBy = shuffled.sortedByDescending { abs(it)  } // 5

    println("Shuffled: $shuffled")
    println("Natural order: $natural")
    println("Inverted natural order: $inverted")
    println("Inverted natural order by absolute value: $descending")
    println("Inverted natural order of absolute values: $descendingBy")
}
```
- 1 创建一个list
- 2 排序，按照自然关系排序
- 3 按照自然关系倒序排序
- 4 按照自然关系倒序排序，通过使用sortedDescending
- 5 按照自然关系倒序排序，通过将每个元素进行取绝对值运算
___

### Map Element Access map元素的访问
kotlin 通过 `[]` 来获取value根据key值，如果获取不到就返回null。`getvalue()` 方法也可以获取当前的value，只不过如果获取不到value，将会throw exception。我们可以通过`withDefault` 创建一个新的map，在这个map中，`getvalue()`将不会抛出异常，取而代之地是将会返回默认值
```kotlin
fun main(args: Array<String>) {

    val map = mapOf("key" to 42)

    val value1 = map["key"]                                     // 1
    val value2 = map["key2"]                                    // 2

    val value3: Int = map.getValue("key")                       // 1

    val mapWithDefault = map.withDefault { k -> k.length }
    val value4 = mapWithDefault.getValue("key2")                // 3

    try {
        map.getValue("anotherKey")                              // 4
    } catch (e: NoSuchElementException) {
        println("Message: $e")
    }


    println("value1 is $value1")
    println("value2 is $value2")
    println("value3 is $value3")
    println("value4 is $value4")
}
```
- 1 返回42
- 2 返回null
- 3 返回4，value 是计算出来地，按照代码是根据key地长度 计算出来地。
- 4 报错，anotherKey 不在map中
___

### zip
将两个集合合并成一个新的集合，默认的合成方式为，结果集中包含的是两个集合的元素按照index 两两组成一个pair，当然你可以按照直接的方式来组成新的数据结构。
结果集的大小是由两个源数据的长度的最小值来决定的
```kotlin
fun main() {

    val A = listOf("a", "b", "c")                  // 1
    val B = listOf(1, 2, 3, 4)                     // 1

    val resultPairs = A zip B                      // 2
    val resultReduce = A.zip(B) { a, b -> "$a$b" } // 3

    println("A to B: $resultPairs")
    println("\$A\$B: $resultReduce")
}
/*
A to B: [(a, 1), (b, 2), (c, 3)]
$A$B: [a1, b2, c3]
*/
```
- 1 定义了两个list
- 2 将两个list 合并成一个pair的list。zip 作为一个中缀方法在这里使用
- 3 将两个list合并成一个list，按照给出的方式合并，在这里，两个参数将被合并成为一个string。结果集也就是一个stirng的list
---

### getOrElse
这个方法提供了一种安全的方式去访问list，不用再担心数组越界的问题，如果访问的不是数组内的index，你可以提供一个默认的值以返回。
```kotlin
fun main() {

    val list = listOf(0, 10, 20)
    println(list.getOrElse(1) { 42 })    // 1
    println(list.getOrElse(10) { 42 })   // 2
}
/*
10
42
*/
```
- 1 打印了第二个元素 10
- 2 第11个元素明显越界了，返回了提供的默认值 42

getOrElse还可以用在map上。
```kotlin
fun main() {

    val map = mutableMapOf<String, Int?>()
    println(map.getOrElse("x") { 1 })       // 1

    map["x"] = 3
    println(map.getOrElse("x") { 1 })       // 2

    map["x"] = null
    println(map.getOrElse("x") { 1 })       // 3
}
/*
1
3
1
*/
```
- 1 打印了默认值1 因为x 不在key的范围内
- 2 打印了 3 因为添加了x的k-v 到map中
- 3 打印了1 应为key为x的value置为null了



## 【译】Kotlin 语法基础 - 代理


### Delegation Pattern 委托模式

kotlin 天然支持 [委托代理模式](https://kotlinlang.org/docs/reference/delegation.html)，不再需要编写一些模板文件。

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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




## 【译】Kotlin 语法基础 - 特殊特性


### Named Arguments

和其他语言（java， c++，之类的）不同的是kotlin提供了一种特性叫命名参数。既可以和别的语言一样按照定义的方法的参数顺序来传递参数，也可以通过名字来传递参数，这样就避免了错误的顺序会带来的错误以及可以更清晰地调用。有些错误是很难以通过编译器来检查出编译错误的。比如这样

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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




## 【译】Kotlin 语法基础 - 特殊类


### Data Class

[Data class](https://kotlinlang.org/docs/reference/data-classes.html?_ga=2.47914596.2133596250.1603679551-1735323035.1603280899) 可以用于创建一个数据类用于存储数据。这些类将会自动提供类似`setter`, `getter`,`copy`,`toString()`,`hashCode()`,`componentN()`方法

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

```kotlin
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

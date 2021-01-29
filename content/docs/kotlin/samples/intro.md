---
title: '【译】Kotlin - 基础介绍'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: "方法、操作符方法、中缀方法、vararg、变量、泛型、继承"
---


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

```
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
> ```
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
> ```
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

```
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

```
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

```
var a: String = "initial"  // 1
println(a)
val b: Int = 1             // 2
val c = 3                  // 3
```

- 1 定义了一个可变变量，并且初始化
- 2 定义了一个不可变变量，并且初始化。相当于java中添加了 final
- 3 定义了一个不可变变量，并且初始化，并且没有明确定义变量名称。类型将由编译器推断，这里编译器推断的是Int

```
var e: Int  // 1
println(e)  // 2
```

- 1 定义了一个变量并且没有对他进行初始化
- 2 这里将会报错，变量必须进行初始化

> 你可以在任何地方进行初始化，只要在read 这个变量之前就可以。这里和java 不同

```
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

```
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

```
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

```
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

```
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

```
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

```
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

```
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

```
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

---
title: '【译】Kotlin 语法基础 - 流程控制'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: "when 、 loop 、 区间"
---


### when

##### when 语句

用来代替广泛使用的 switch 关键字，Kotlin 提供了一种更加灵活和简洁的when 结构。when可以被用来当作一个语句，也可以用来当作一个表达式

```
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

```
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

```
val cakes = listOf("carrot", "cheese", "chocolate")

for (cake in cakes) {                               // 1
    println("Yummy, it's a $cake cake!")
}
```

- 1 将会遍历每个参数

##### while 和 do-while

`while` 和 `do-while` 在kotlin的使用方式与大多数语言一致

```
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

```
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

```
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

```
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

```
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

```
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

```
fun max(a: Int, b: Int) = if (a > b) a else b         // 1

println(max(99, -42))
```

- 1  返回 a或者b

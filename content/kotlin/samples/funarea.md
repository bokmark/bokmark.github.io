---
title: '【译】Kotlin 语法基础 - 作用域方法'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: 'let 、 run 、 with 、 apply 、 also'
---

### let

let 是一个kotlin 的标准库，可以用于实例检查和null检查，当let被调用再一个实例上的时候，你可以通过it 来再作用域内访问这个实例。作用域内的最后一句表达式，将会作为返回值返回

```
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

```
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

```
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

```
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

```
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

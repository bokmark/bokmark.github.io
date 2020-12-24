---
title: 'Kotlin 进阶知识'
date: 2020-11-30T19:30:08+10:00
draft: false
weight: 2
---

- 顶级属性和顶级方法

  ```
  val PI = 3.14
  var x = 0

  fun incrementX() {
      x += 1
  }

  fun main() {
      println("x = $x; PI = $PI")
      incrementX()
      println("incrementX()")
      println("x = $x; PI = $PI")
  }
  ```
  定义在class 外面的属性 可以理解为 java中public 静态属性，可以被所有地方使用，只需要import 即可
  方法同理。

- 幕后属性
  ```Kotlin
  private var _table: Map<String, Int>? = null
  public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 类型参数已推断出
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
  ```
  以上代码中`table` 属性就是幕后属性。当我们访问`table`属性的时候其实访问的是 get() 方法，get方法中访问了`_table`属性，这就造成了 `table` 属性其实并不会被访问到的情况

  ```kotlin
  class demo(var name: String) {
    val size: Int
        get() {
            return name.length
        }
  }
  fun main() {
    val a = demo("sq")
    println(a.size)

    a.name = "111123"
    println(a.size)
  }
  /** result
  2
  6
  */

  ```
  从这里我们可以看到访问size 其实就是在访问get方法，
  - *我们可以认为 size 属性就是get方法的别名，val 修饰属性 只是限制了set方法的出现，var修饰属性可以添加set方法而已*

- `?:`
  ```
  val isEmpty = ...
  val a = isEmpty ?: c
  ```
  这里如果isEmpty 为空，将会默认执行 c。 c可以是一个变量，也可以是一个语句。其实这就是kotlin中的特性，{} 中的最后一个语句将作为返回值

- reified -- 使抽象的东西更加具体或真实
  [可以查看这篇文章](https://juejin.cn/post/6844903833596854279)
  ```
  inline fun <reified T : Activity> Activity.startActivity() {
    startActivity(Intent(this, T::class.java))
  }

  startActivity<NewActivity>()
  ```

- as? 安全的类型转化。 a as b 如果a 为空则返回null 否则类型转化
  ```
  val nullorNot = ...
  val nullorNot as? Abc
  ```
- 基本数字类型
|类型 |	大小（比特数）| 	最小值 |	最大值|
|--|--|--|--|
|Byte |	8 |	-128 |	127
|Short |	16 |	-32768 |	32767
|Int |	32 |	-2,147,483,648 (-231) |	2,147,483,647 (231 - 1)
|Long |	64 |	-9,223,372,036,854,775,808 (-263) |	9,223,372,036,854,775,807 (263 - 1)

  默认数字类型不超过int的范围推断为Int。可以通过在数字后面添加L 来显式地表明为Long型
- sam 转化
  single abstract method。只有一个方法的 接口可以通过kotlin 的转化成更加简洁的写法。
- kotlin 集合分为 可变集合 和不可变集合

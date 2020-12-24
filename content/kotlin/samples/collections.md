---
title: '【译】Kotlin 语法基础 - 集合'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 10
summary: "list 、 map 、 set、fliter、any、all、none、first、last、partition"
---

### [List](https://kotlinlang.org/docs/reference/collections-overview.html)
list 是一个有序的集合。在kotlin中 list 可以是可变的 `MutableList ` 也可以是只读的 `List `。在创建list的时候，使用标准类库里的 `listOf()` 创建只读的不可变的 `List`，使用 `mutableListOf()` 创建可变的list。为了避免不必要的修改，你可以用List的引用来指向可变list， 这将避免错误的修改。
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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

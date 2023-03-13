---
author: Bokmark Ma
title: Compose 性能优化
date: 2022-08-29T17:25:16+08:00
slug: compose/performance
description: Compose 性能优化
draft: true
categories:
    - Compose
tags:
    - compose
---

# Compose 优化

> [链接](https://www.youtube.com/watch?v=EOQB8PTLkpY&ab_channel=AndroidDevelopers)
{{< youtube EOQB8PTLkpY >}}

## 视频重要内容摘录

### 正确的配置【Configuration】
正确的设置compose的配置可以提高compose的性能表现

```groovy
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```
- *确保你发现的性能问题是在R8的release模式下发现的。*  
在debug模式下，android会关闭某些优化功能，以提高调试体验。

### 一些简单的例子【Gotchas】
#### remember
**先看下这段代码**
```kotlin
// 这是一段通讯录的demo代码
@Composable
fun ContactList(
    contacts: List<Contact>,
    comparator: Comparator<Contact>,
    modifier: Modifier = Modifier
) {
    LazyColumn(modifier) {
        // 这是错误示范，请不要这么做
        items(contacts.sortedWith(comparator)) { contact ->
            ...
        }
    }
}
```

**这段代码问题是什么？**
- 当`contacts`修改数据的时候由于`sortedWith`将会生成一个新的list从而导致items的参数变化，这将导致`LazyColumn`会在数据修改时重新刷新整个list。  

**针对这个问题该如何解决？**

```kotlin
@Composable
fun ContactList(
    contacts: List<Contact>,
    comparator: Comparator<Contact>,
    modifier: Modifier = Modifier
) {
    val sortedContacts = remeber(constacts, comparator) {
        constacts.sortedWith(comparator)
    }
    LazyColumn(modifier) {
        items(sortedContacts) { contact ->
            ...
        }
    }
}
```
使用`remember` 来缓存一些消耗大的操作。  
**先来看看remember的定义**
```kotlin
/**
 * Remember the value returned by [calculation] if [key1] is equal to the previous composition,
 * otherwise produce and remember a new value by calling [calculation].
 * 
 * 如果[key1]和上次重组时的值不一样，将会调用[calculation]产生一个新值，并且返回这个新值。
 * 如果[key1]和上次重组时的值一样，将会返回上次记忆好的值。
 */
@Composable
inline fun <T> remember(
    key1: Any?,
    crossinline calculation: @DisallowComposableCalls () -> T
): T {
    return currentComposer.cache(currentComposer.changed(key1), calculation)
}

```
通过添加`remember`，避免了每次list的修改会导致的重组，提升了compose的性能。

**更优秀的解决方案**  
将list的数据来源以及sort操作，放置到viewmodel或者datasource中。

#### Key
还是刚刚的例子，我们优化了sort的操作，将其放置到viewmodel中之后，例子如下：
```kotlin
@Composable
fun ContactList(
    contacts: List<Contact>, 
    modifier: Modifier = Modifier
) { 
    LazyColumn(modifier) {
        items(contacts) { contact ->
            ...
        }
    }
}
```
**还可以如何改进？**
这就引出了 lazylist的另一个参数 *key*
```kotlin

/**
 * 在lazylist中添加list
 *
 * @param items 数据list

 * @param key 一个根据item产生唯一key值的工厂。比如注意key必须是唯一的。key必须可以被保存到Android的bundle种。null值也将是一种key值。 如果你传入的key是特殊的当前列表的position，
 factory of stable and unique keys representing the item. Using the same key
 * for multiple items in the list is not allowed. Type of the key should be saveable
 * via Bundle on Android. If null is passed the position in the list will represent the key.
 * When you specify the key the scroll position will be maintained based on the key, which
 * means if you add/remove items before the current visible item the item with the given key
 * will be kept as the first visible one.

 * @param contentType 
 a factory of the content types for the item. The item compositions of
 * the same type could be reused more efficiently. Note that null is a valid type and items of such
 * type will be considered compatible.


 * @param itemContent the content displayed by a single item
 */
inline fun <T> LazyListScope.items(
    items: List<T>,
    noinline key: ((item: T) -> Any)? = null,
    noinline contentType: (item: T) -> Any? = { null },
    crossinline itemContent: @Composable LazyItemScope.(item: T) -> Unit
) = items(
    count = items.size,
    key = if (key != null) { index: Int -> key(items[index]) } else null,
    contentType = { index: Int -> contentType(items[index]) }
) {
    itemContent(items[it])
}
```



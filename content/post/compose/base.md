---
author: Bokmark Ma
title: Compose 原理
date: 2022-08-10
description: Compose 原理
draft: true
slug: compose/base
categories:
    - Compose
tags:
    - compose
---

# Compose 原理探索

## 什么是Compose？为什么采用Compose？

- 使用Kotlin 开发的，用于构建原生的Android界面的新工具包。

- 为什么使用 Compose
    - 更少的代码。更少的代码就以为着更少的bug，更高的可维护性，更高效的开发方式。
    - 当ui和代码使用同一种代码时，更方便的追踪代码。
    - 更简洁的ui框架，易于理解以及维护。
    - 更加直观，与Activity或者fragment无关的ui小组件，确保他更加易于维护和使用。
    - 与现有的view体系融合，可以更方便的接入开发。

## compose 各个方面的文档

https://developer.android.com/jetpack/compose/documentation

## Compose的编程思想

### Compose，声明式UI

#### 什么是声明式UI，什么是命令式UI

- 声明式UI：你只需要声明UI是什么样子的就可以，不需要你去手动更新。
    - 声明性界面模型，该模型大大简化了与构建和更新界面关联的工程任务。该技术的工作原理是在概念上从头开始重新生成整个屏幕，然后仅执行必要的更改。
    - 重新生成整个屏幕所面临的一个难题是，在时间、计算能力和电池用量方面可能成本高昂。为了减轻这一成本，Compose 会智能地选择在任何给定时间需要重新绘制界面的哪些部分
- 命令式UI：由你下命令来手动更新UI。
- [引用1](quote/compose)

> 在xml的ui方案中，我们通过inflate xml文件来构建viewTree，每个view维护自己的状态，代码逻辑来控制每个view的状态。

> 但是在Compose的方式中，Composable方法是无状态的，我们通过传递不同参数来调用同一个Composable方法以打到修改ui的界面。

在许多面向对象的命令式界面工具包中，您可以通过实例化 widget 树来初始化界面。您通常通过膨胀 XML 布局文件来实现此目的。每个 widget 都维护自己的内部状态，并且提供 getter 和 setter 方法，允许应用逻辑与 widget 进行交互。

![Compose 界面中数据从高级对象向下流向其子级的图示。](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-data.png)


*图 2.* 应用逻辑为顶级Composable函数提供数据。该函数通过调用其他Composable函数来使用这些数据描述界面，将适当的数据传递给这些Composable函数，并沿层次结构向下传递数据。

当用户与界面交互时，界面会发起 onClick 等事件。这些事件应通知应用逻辑，应用逻辑随后可以改变应用的状态。当状态发生变化时，系统会使用新数据再次调用Composable函数。这会导致重新绘制界面元素，此过程称为“重组”。

![说明界面元素如何通过触发由应用逻辑处理的事件来响应交互的图示。](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-events.png)

*图 3.* 用户与界面元素进行了交互，导致触发一个事件。应用逻辑响应该事件，然后系统根据需要使用新参数自动再次调用Composable函数。

### Compose 的优点

#### Kotlin语言
由于Composable 是用Kotlin写的，kotlin的所有语言优势，Compose都具备。
比如根据状态做if判断 for循环等。

#### Compose 重组
由于Compose 是一个从上到下的整体树，点击事件等用户反馈，会导致大量的计算，为了优化这种情况，Compose 通过智能重组方案来解决。

> 重组是指当函数的输入更改时，会发生再次调用Composable函数的情况。当 Compose 根据新输入重组时，它仅调用可能会被改变的的 Composable函数。通过跳过所有未更改参数的Composable函数的，重组才可能高效。

而以下几点需要注意
- Composable函数可以按任何顺序执行。
- Composable函数可以并行执行。
- 重组会跳过尽可能多的Composable函数。
- 重组是乐观的操作，可能会被取消。
- Composable函数可能会像动画的每一帧一样非常频繁地运行。
而根据这几天，你需要在你的代码里保持以下几点，以避免出现问题

##### Composable函数可以并行执行
Composable函数可以并行执行，以利用设备的多个cpu。而这就意味着Composable函数可能会在一个后台的线程池里运行，如果你低啊用viewmodel的方法，那么将发生意外的错误。
Composable函数并不是线程安全的方法，如果在Composable函数中做一些计算，可能导致异常的问题。

##### 重组将会尽量跳过能跳过的内容
```Kotlin
/**
 * Display a list of names the user can click with a header
 */
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
        // this will recompose when [header] changes, but not when [names] changes
        Text(header, style = MaterialTheme.typography.h5)
        Divider()

        // LazyColumn is the Compose version of a RecyclerView.
        // The lambda passed to items() is similar to a RecyclerView.ViewHolder.
        LazyColumn {
            items(names) { name ->
                // When an item's [name] updates, the adapter for that item
                // will recompose. This will not recompose when [header] changes
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

/**
 * Display a single name the user can click.
 */
@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit) {
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```
compose 在header改变时，可能会只修改`Text(header)` 而不修改顶层的Column，同样的，names的元素的修改而不是实例的修改，可能会导致 LazyColumn 的变化被跳过。

> 在Compose中的所有修改，都应该在回调中去修改

##### 重组可取消
当新的参数来的时候，之前的重组可能会被取消，重新开始新的重组

所以，不可以在Compose函数中进行一些带有副作用的炒作。

##### Composable函数，可能会被非常频繁的运行
可以将Composable函数当成是老的ui方案里的onDraw函数，一些函数炒作需要放到到外部去操作。
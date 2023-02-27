---
author: Bokmark Ma
title: "Compose 引用1 - 声明式UI"
date: 2022-08-29T17:25:16+08:00
slug: "compose"
description: Compose 系列 引用1 - 声明式UI
categories:
    - Quote
tags:
    - Compose
---

# 声明式 UI；更简单的自定义；实时的、带交互的预览功能；还有更强的性能和功能。这就是 Android 官方全新推出的 UI 框架——Jetpack Compose。

大家好，我是扔物线朱凯。

2019 年中，Google 在 I/O 大会上公布了 Android 最新的 UI 框架：Jetpack Compose。Compose 可以说是 Android 官方有史以来动作最大的一个库了。它在 2019 年中就公布了，但要到今年也就是 2021 年才会正式发布。这两年的时间 Android 团队在干嘛？在开发这个库，在开发 Compose。一个 UI 框架而已，为什么要花两年来打造呢？因为 Compose 并不是像 RecyclerView、ConstraintLayout 这种做了一个或者几个高级的 UI 控件，而是直接抛弃了我们写了 N 年的 View 和 ViewGroup 那一套东西，从上到下撸了一整套全新的 UI 框架。直白点说就是，它的渲染机制、布局机制、触摸算法以及 UI 的具体写法，全都是新的。

---

## Compose 的写法
Compose 从一出现，最受到官方推崇以及关注者赞扬的就是它实现了声明式 UI，说它比我们传统写法的「命令式 UI」怎么怎么好——我们传统的 View 和 ViewGroup 那一套系统的写法叫「命令式」。但是对于大多数 Android 开发者来说，我们的第一个问题就是：什么是「声明式 UI」？

在讲「声明式 UI」之前，我们先看一下 Compose 的代码长什么样。Compose 是用 Kotlin 来写的，它的每个控件都是一个函数调用。比如你要显示一块文字，你就这么写：

```Kotlin
Text("Hello")
```

看起来好像只是调用构造函数创建了一个对象，但这么写就已经可以显示出一块文字来了。

另外——这其实也并没有创建对象，这个 `Text()` 也不是一个构造函数，而是一个普通函数。
```Kotlin

Text("Hello")

...

@Composable
fun Text(...) {
    ...
}
```

普通函数用大写开头干嘛？很简单，为了辨识度。Compose 规定了这种大写开头的命名方式，这样我们就能一眼认出来：哦，这是个 Compose 的函数——或者用更官方的叫法：这是一个 Composable。

到这儿有人可能就会想：这个 `Text()` 它实质上是个什么？是个 `TextView` 吗？不是的。刚才我说过一次，Compose 的渲染机制、布局机制、触摸机制全都是新写的，所以这个 `Text()` 的底层不是 `TextView`，也不是任何一个原生控件，而是直接调用了更下层的绘制 API，也就是 `Canvas` 那一套东西。同理，Compose 里的各个组件，都是独立的新实现。

好继续说。一个函数调用是一个组件；两个函数调用就是两个组件；
```Kotlin
Text("Hello")
Image()
多个函数组合起来，就是一个完整的界面：

Column {
    Text("Hello")
    Image()
}
```
这，就是 Compose 的写法。看完它的写法，我们就可以回到刚才的问题：什么是「声明式 UI」？这段代码怎么就「声明式」了？它和我们一直以来的写法有什么区别？

首先，我们一般怎么写 UI 的？xml 文件，对吧？比如这个界面，上下排列的一块文字和一个图片，它的等价传统写法是这样的：
```XML
<!-- 代码经过一定简化 -->
<LinearLayout>
  <TextView android:text="Hello" />
  <ImageView />
</LinearLayout> 
```
一个 `LinearLayout`，里面包着一个 `TextView` 和一个 `ImageView`。

看了以后什么感觉？大同小异是吧？除了名字换换、格式变变，大体上是一样的。对吧？

那为什么左边就叫命令式，右边就叫声明式呢？xml 命令谁了？以及，右边这写法怎么就更优秀了？我为什么要学一个看起来并没有什么本质区别的新写法来为难自己？

其实所谓「声明式 UI」，指的是你只需要把界面给「声明」出来，而不需要手动更新。关键在于「不需要手动更新」。比如左边这个布局里的 TextView，如果它对应的数据改变了，我要怎么把新的文字更新到它？很简单：findViewById()、setText() 对吧？
```Kotlin
findViewById()
setText()
```
而如果用 Compose 呢？怎么更新？不用更新。因为 Compose 的界面会随着数据自动更新。

Compose 会对界面中用到的数据自动进行订阅——不管是字符串还是图像还是别的什么，Compose 全部能够自动订阅——这样当数据改变的时候，Compose 会直接把新的数据更新到界面。
```Kotlin
var text = "Hello"

...

Column {
    Text(text)
    Image()
}
这个「自动订阅」的功能很容易使用，你只要在初始化的时候加上一个 by mutableStateOf() ，剩下的全都由 Compose 自动搞定。

var text by mutableStateOf("Hello")

...

Column {
    Text(text)
    Image()
}
```

这个神奇的功能是利用 Kotlin 的 Property Delegation 属性委托来实现的。这也在一定程度上回答了一个问题： 为什么 Compose 只能用 Kotlin 写，而不能用 Java？因为它用了大量的 Kotlin 特性，而这些特性用 Java 不能简单实现。注意，虽然 Kotlin 和 Java 是兼容的，Kotlin 能做到的事 Java 也能做到，但是有些东西它「不能简单实现」就约等于不能实现了，因为不实用啊！对吧？所以 Android 自称永远不放弃对 Java 的支持，他们就这么一说，你就这么一听，不要真的就不学 Kotlin，不然会越来越难受。你看这 Compose 不是已经在逼着我们用 Kotlin 了吗？

好拐回来，这就是所谓的「声明式 UI」：你只要声明界面是什么样子，不用手动去更新，因为界面会自动更新。而传统的写法里，数据发生了改变，我们得手动用 Java 代码或者 Kotlin 代码去把新数据更新到界面。你给出详细的步骤，去命令界面进行更新，这就是所谓的「命令式 UI」。

那么现在我们再往回拐：传统的 xml 写法和 Compose 的 Kotlin 写法，为什么一个是「命令式」，一个是「声明式」？这个问题其实本身就是错的。单单一段 xml 代码并不能称作是命令式 UI。传统写法的「命令式」并不在于 xml 部分，而在于 Java 部分：Java 代码去指挥、去命令界面更新，这才是「命令式」的含义所在；而 Compose 通过订阅机制来自动更新，所以不需要做这种「命令」，所以是「声明式」。

所以你看，不管是声明式还是命令式，跟 xml 和 Kotlin 是无关的，它们并不是语言角度的定义，也不是写法角度的定义，而是——功能角度。一个 UI 框架，如果可以让开发者只声明出界面的样子，而不用去写各种界面更新的代码，它就是一个声明式的 UI 框架。换句话说，如果 Android 可以让我们用 xml 写的界面也和数据做关联，让界面自动更新而不需要开发者手写更新代码，那么它就也是声明式 UI。声明式 UI 是一种强大的功能，而不是一种优秀的代码风格。

哎？数据和界面做关联，界面跟着数据自动更新，这不就是数据绑定吗？Android 已经有这样的官方库了啊！就叫 Data Binding，是吧？我用它不就得了，为什么要费这么大劲去用 Compose 呢？

首先，对！Data Binding 和 Compose 本质上都是通过界面对数据进行订阅来实现了界面的自动更新，但！它们是有关键区别的。区别就在于，Data Binding 通过数据更新的只能是界面元素的值，而 Compose 可以更新界面中的任何内容，包括界面的结构。比如你用一个 Boolean 类型的变量控制界面中某个元素是否显示，

```Kotlin
var text = ...
var showImage = ...
Column {
    Text(text)
    if (showImage) {
        Image()
    }
}
```
当你把变量的值从 true 变成 false 的时候，
```Kotlin
var text = ...
var showImage = ...
Column {
    Text(text)
    if (showImage) {
        Image()
    }
}

...
showImage = false

```

这个元素会从界面中完全消失，就像从来没有出现过一样，而不是用 setVisibility(GONE) 这种方式从视觉上隐藏。这两种策略看起来好像区别不大，那是因为我举的例子简单，实际上这是一种机制的改变，而这种机制的改变给界面开发带来的灵活性和性能的提升是非常大的。你想一下，是不是？
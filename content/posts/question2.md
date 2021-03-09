---
title: '面试题'
date: 2020-02-18
weight: 1
draft: true
tags: ["面试"]
categories: ["面试"]
menu: "main"
summary: "面试题合集"
---

# 面试题

## 1、泛型和多态的意义和使用场景？
java中多态存在的意义：
　　1.可替换性（substitutability）。多态对已存在代码dao具有可替换性。例zhuan如，多态对圆shuCircle类工作，对其他任何圆形几何体，如圆环，也同样工作。
　　2.可扩充性（extensibility）。多态对代码具有可扩充性。增加新的子类不影响已存在类的多态性、继承性，以及其他特性的运行和操作。实际上新加子类更容易获得多态功能。例如，在实现了圆锥、半圆锥以及半球体的多态基础上，很容易增添球体类的多态性。
　　3.接口性（interface-ability）。多态是超类通过方法签名，向子类提供了一个共同接口，由子类来完善或者覆盖它而实现的。
　　4.灵活性（flexibility）。它在应用中体现了灵活多样的操作，提高了使用效率。
　　5.简化性（simplicity）。多态简化对应用软件的代码编写和修改过程，尤其在处理大量对象的运算和操作时，这个特点尤为突出和重要
## 2、jvm数据区域？都存储哪些数据？内存泄漏发生在哪些区域？
## 3、软引用和弱引用的区别？使用场景？

## 4、HashMap的put()方法，时间复杂度？怎样使得HashMap线程安全？ConcurrentHashMap线程安全是怎样实现的？
最优情况，hash不碰撞，O(1)，典型情况，近似是O(1)，因为几乎没有碰撞，最坏情况，O(N)，也就是所有的hash都一样，那么退化为线性查找。
## 5、synchronized和ReentrantLock的区别？使用场景？

什么时候选择用 ReentrantLock 代替 synchronized？
既然如此，我们什么时候才应该使用 ReentrantLock 呢？答案非常简单 —— 在确实需要一些 synchronized 所没有的特性的时候，比如时间锁等候、可中断锁等候、无块结构锁、多个条件变量或者轮询锁。 ReentrantLock 还具有可伸缩性的好处，应当在高度争用的情况下使用它，但是请记住，大多数 synchronized 块几乎从来没有出现过争用，所以可以把高度争用放在一边。我建议用 synchronized 开发，直到确实证明 synchronized 不合适，而不要仅仅是假设如果使用 ReentrantLock “性能会更好”。请记住，这些是供高级用户使用的高级工具。（而且，真正的高级用户喜欢选择能够找到的最简单工具，直到他们认为简单的工具不适用为止。）。一如既往，首先要把事情做好，然后再考虑是不是有必要做得更快。

## 6、什么是原子性操作？怎么保证原子性操作？
## 7、A-->B-->B中加载一个Fragment，生命周期
## 8、MVC和MVP，减少MVP接口的方法
## 9、service的生命周期
## 10、onSaveInstanceState()什么时候会被调用
1.从activity A中启动一个新的activity时；
2.屏幕方向切换时，例如从竖屏切换到横屏时；
注意：在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行。
3.按下电源按键（关闭屏幕显示）时；
4.当用户按下HOME键，回到桌面时。

总结 当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用。另外，onSaveInstanceState()会在onPause()或onStop()之间执行，onRestoreInstanceState()会在onStart()和onResume()之间执行。

屏幕方向切换时，Activity被系统销毁后重新建立，此时会调用onSaveInstanceState方法保存数据。方法执行过程为：
onPause –>onSaveInstanceState–>onStop–>onDestory–>onCreate(切换屏幕后重新创建Activity时调用的onCreate方法)–>onStart–>onRestoryInstanceState–>onResume。
## 11、RecyclerView和ListView的区别？
## 12、ConstraintLayout的优势？和RelativeLayout的区别？
- 实现复杂的页面 也可以减少层级。
- 可以直接用编辑器快速搭建出页面，
## 13、优化网络请求
## 14、报文
## 15、HTTPS和HTTP的区别？SSL加密过程？
## 16、开发中怎么限制三方库的大小？
- https://blog.csdn.net/earbao/article/details/79677034
## 17、进程间通信有哪几种方式？
- socket、共享内存、管道、信号量、binder
## 18、AIDL

①腾讯音乐
1、性能优化，Apo启动加速优化（音视频）
2、Java crash定位
3、分析举例
4、App启动加速
②腾讯PCG-腾讯视频Android笔试题
1、Java class文件的构成
2、Java反射效率低的原因及解决方案
3、message的同步异步处理 及消息屏障
4、不属于thread的方法   spak  join  sleep
5、爬楼梯，n阶楼梯，只能走1步或2步，共需要几步
6、找出一组数中 2的个数
③腾讯音乐
1、算法
设计⼀个复⽤池
⽤最少的空间复杂度 将⼀个数组正数和负数分开
给定⼀个数请给出⼀个⽐它⼤的最⼩回⽂数
 
https 加不加密Url
intent中如果放进数据超过2m 会发⽣什么
如果要往数据库中放⼊⼆进制数据怎么办
储存的⽂件有可能被⽤户修改怎么处理
android如何捕获卡顿 如果让你设计如何判断主线程卡顿 如何获取主线程卡顿时的堆栈
客户端如何获取到jsBridge中传过来的对象
2、Java与C++如何相互调⽤？
Android中如何⾃定义⼀个View，可以允许我们像使⽤系统⾃带的View⼀样使⽤它？（⽐
如在布局⽂件中使⽤并定制）
3、编程 - 画家⼩Q 画家⼩Q⼜开始他的艺术创作。⼩Q拿出了⼀块有NxM像素格的画板, 画板初始状态是空⽩ 的,⽤’X’表示。
⼩Q有他独特的绘画技巧,每次⼩Q会选择⼀条斜线, 如果斜线的⽅向形如’/‘,即斜率为1,⼩Q 会选择这条斜线中的⼀段格⼦,都涂画为蓝⾊,⽤’B’表示;如果对⻆线的⽅向形如’’,即斜率为-1, ⼩Q会选择这条斜线中的⼀段格⼦,都涂画为⻩⾊,⽤’Y’表示。
如果⼀个格⼦既被蓝⾊涂画过⼜被⻩⾊涂画过,那么这个格⼦就会变成绿⾊,⽤’G’表示。
⼩Q已经有想画出的作品的样⼦, 请你帮他计算⼀下他最少需要多少次操作完成这幅画。输 ⼊描述
每个输⼊包含⼀个测试⽤例。
每个测试⽤例的第⼀⾏包含两个正整数N和M(1 <= N, M <= 50), 表示画板的⻓宽。
接下来的N⾏包含N个⻓度为M的字符串, 其中包含字符’B’,’Y’,’G’,’X’,分别表示蓝⾊,⻩⾊,绿⾊, 空⽩。整个表示⼩Q要完成的作品。
输出描述
输出⼀个正整数, 表示⼩Q最少需要多少次操作完成绘画
示例1
输⼊
4 4
YXXB
XYGX
XBYY
BXXY
输出3
说明
XXXX
XXXX
XXXX
XXXX
→
YXXX
XYXX
XXYX
XXXY
→
YXXB
XYBX
XBYX
BXXY
→
YXXB
XYGX
XBYY
BXXY
请描述Touch Event在Android系统中的分发流程
如何分析Android App发⽣了内存泄漏？
TCP、UDP传输流媒体分别有哪些优缺点？
Java语⾔中synchronized关键字有哪些⽤法，区别时候什么；在进⾏多线程开发时如何避
免死锁？产品上线了如何检测死锁
kotlin对⽐java有哪些优势
协程如何实现调度的 协程如何取消
liveData 有哪些优势
4、场景题
往链表⾥⾯插⼊100个数 要求有正有负范围不限
移除⾥⾯的负数
利⽤多线程加速移除负数的过程
⼀个布局可以同时⽤ LinearLayout 和 RelativeLayout 实现从多个⻆度分析应该选⽤哪⼀个
https 的通讯过程 如何检查ca证书的正确性
④腾讯音乐
1.MVVM
2.我http状态码
3.handler原理
4.性能优化
5.recylerview源码
6.binder机制
7.分享在项目中的技术难点，以及如何解决的
8.如何做线上性能监控
9.事件分发流程
10.如何处理事件冲突

 

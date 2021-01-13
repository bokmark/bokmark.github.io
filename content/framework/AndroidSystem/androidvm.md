---
title: 'android vm'
date: 2018-11-28T15:14:39+10:00
weight: 1
summary: "android vm 和 java vm 的区别"
---

- Java vm
  基于栈的虚拟机，每一个运行的线程，都有一个独立的栈，每一个方法调用都会产生一个栈帧。当前线程运行的方法的栈帧在当前虚拟机栈的栈顶。所有的运算都通过栈帧中的操作数栈来炒作。
  每一个栈帧中包含 本地变量表，操作数栈，动态链接，返回地址。
- Android vm
  基于寄存器的虚拟机，取消了操作数栈。本地变量表使用虚拟的寄存器来代替，相应的字节码指令也将会变少，减少了数据在操作数栈和本地变量表中的移动。相当于将操作数栈和本地变量表合并在寄存器中。

[dalvik](https://source.android.google.cn/devices/tech/dalvik?hl=zh-cn)

- jit 即时编译

- dalvik与 art的区别
  - dalvik：解释执行dex字节码，结合jit将部分热点代码即时进行编译或者优化成本地机器码。
  - art：安装的时候将dex 代码转化成本地机器码
- dalvik与 art的 安装的区别
  - dalvik 是通过 dexopt 将apk中的dex文件提取放到存储中
  - art 是通过dex2oat 将dex文件提取出来 编译成当前设备的机器码 aot （预编译）
- art的进化，从Android N 开始使用混编
  - 安装的时候将dex放到本地。
  - 先运行的时候先使用jit做一个编译，并且将编译的部分保存到profile。
  - 当空闲的时候将profile文件中保存的代码进行oat编译成本地代码。
  - 运行时查找本地代码，如果有就还是用本地代码 如果没有则使用jit编译并且保存到profile

- 类加载机制
  - ClassLoader
    - BootClassLoader // 加载基础的framework中的类
    - BaseDexClassLoader // 加载apk中的dex的类，中间有一个 dexpathlist
      - PathClassLoader // Android apk的类加载机制
      - DexClassLoader // 动态加载其他的dex文件

- PathClassLoader 源码解析

  - 双亲委派机制
    - 优先是由parent classloader 来进行加载，如果parent classloader 加载失败，再由自己进行加载。如果失败抛出classnotfoundexception



热修复等问题遇到 机器码怎么办？

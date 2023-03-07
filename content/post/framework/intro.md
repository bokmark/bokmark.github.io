---
author: Bokmark Ma
title: "Android 基础篇"
date: 2021-10-26T00:16:18+08:00 
slug: fwk/intro
description: 如何变易获取代码
draft: true
categories:
    - Android
tags:
    - Framework
    - Android
---

# Android Framework 基础篇
> [AMS](/p/f/ams)

## 下载安装编译android的源码

从这篇官方的文档 [搭建构建环境](https://source.android.com/setup/build/initializing) 可以看出，我们已经无法再在Mac或者window上进行Android系统的编译。

### 所以第一步，我们需要安装一个linux系统。

你可以安装双系统，也可以安装虚拟机。鉴于m1强大的运算能力，我这里推荐使用虚拟机vmware来安装linux。

#### VM WARE

[这是一个VMWare 不正经的下载地址](https://www.macwk.com/soft/vmware-fusion)
官方推荐使用ubuntu 14.04来进行编译,我也不知道为什么，那就下这个版本吧！[这是链接](https://releases.ubuntu.com/14.04/ubuntu-14.04.6-desktop-amd64.iso)

#### multipass

```
multipass launch --name aosp 18.0
```


https://multipass.run/
http://code.newban.cn/253.html




### 第二步，搭建构建环境

这一步主要查看官方文档 [https://source.android.com/setup/build/initializing](https://source.android.com/setup/build/initializing)

```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip
```

https://source.android.com/setup?hl=zh-cn

# 


```bash

python3 ~/bin/repo init --depth=1  -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r3
```







https://lug.ustc.edu.cn/wiki/mirrors/help/aosp/








## 阅读资料

https://yuedu.163.com/book_reader/807d083015804f29b194c8fe5bfb6732_4

https://www.jianshu.com/p/47eca41428d6

https://weread.qq.com/web/reader/e3d32fb0593388e3dde8006kc81322c012c81e728d9d180

https://weread.qq.com/web/reader/3ee32e60717f5af83ee7b37
---
title: 'Java 对象头的内容'
date: 2018-11-28T15:14:39+10:00
weight: 4
summary: "摘抄自 《深入理解Java虚拟机 -- JVM高级特性与最佳实践》"
---

> 转自 这篇文章 修改了其中的几处错误 https://blog.csdn.net/lkforce/article/details/81128115


Java对象保存在内存中时，由以下三部分组成：

>    1，对象头
>    2，实例数据
>    3，对齐填充字节

### 对象头的内部组成

Java 的对象头的数据
>    1，Mark Word 32bit
>    2，指向方法区中的指针
>    3，数组长度（只有数组对象才有）

##### mark word 长度32bit



![mark word](/images/object_head.png)todo
- 无锁状态下，25个bit的hashcode，4个bit的分代年龄，1个bit的偏向锁标示位 固定为0 ，2个bit的锁标记位 固定为01
- 偏向锁状态下，23个bit的线程ID，2个bit的epoch，4个bit的分代年龄，1个bit的偏向锁标记位固定为1 ，2个bit的锁标记位 固定为01

-- 当从偏向锁升级到 轻量级锁的机制，todo 翻书

*在大量并发的情况下应该弃用偏向锁*

- 轻量级锁状态下，30个bit指向栈中的markword的备份 2个bit的锁标记位固定为00
- 重量级锁状态下，30个bit指向mutex的锁的指针，2个bit的锁标记为固定为10
- gc的时候，30个bit为空，2个bit的锁标记位固定为11

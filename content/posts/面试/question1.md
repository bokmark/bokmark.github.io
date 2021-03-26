---
title: '面试题 题目以及 我自己的答案'
date: 2020-02-18
weight: 1
tags: ["面试", "答案"]
categories: ["面试"]
draft: true
menu: "main"
summary: "面试题合集"
---
# 面试题目以及答案第一部分

## listview与recyclerview的区别
listview
- 二级缓存，缓存的是view：可见view 和不可见 view
- recyclerbin 来实现两级缓存

recyclerview
- 四级缓存，缓存的是viewholder：
  - mChangeScrap 和 mAttchedScrap 屏幕内的 viewholder
  - mCachedScrap 屏幕之外的viewholder
  - mViewCacheExtension 自定义缓存
  - RecycledViewPool viewholder 缓存池
- recycler 来实现四级缓存
- https://www.jianshu.com/p/257c279a3493

## 事件分发流程
[事件分发好文](https://blog.csdn.net/json_it/article/details/100715898)
![a](https://img-blog.csdnimg.cn/20190912083811782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pzb25faXQ=,size_16,color_FFFFFF,t_70)
当输入设备可用时，比如触屏，Linux内核会在/dev/input中创建对应的设备节点，输入事件所产生的原始信息会被Linux内核中的输入子系统采集，原始信息由Kernel Space的驱动层一直传递到User Space的设备节点.

IMS所做的工作就是监听/dev/input下的所有的设备节点，当设备节点有数据时会将数据进行加工处理并找到合适的Window，将输入事件派发给他。

window 再通过两个线程 一个读取 一个分发。读取到input事件后，再将事件传递给activity

```JAVA
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) { // phonewindow 的 superdispatchtouchevent
        return true;
    }
    return onTouchEvent(ev);
}
```

---
title: 'Activity launchMode'
date: 2018-11-28T15:14:39+10:00
weight: 9
summary: "Activity 的 launchemode"
---

# Activity的LaunchMode
描述几个概念，便于理解
A ：Activity A
B ：Activity B
T1：Task1，A 所在的Task
T2：待说明

## 单纯只看 xml的情况下
最近任务中，一个app 只能显示一个task。
如果不点击最近任务或者home键，task将按照启动的顺序一直折叠。
点击最近任务按钮之后，task会被展开。不再将task堆叠在一起。
前台task，

## B的启动模式：默认即standard（这个模式启动不会去考虑taskaffinity）
- 现象：
  在当前task的栈顶即A所在的Task T1，新加入一个B。
- 记忆要点：
  启动这个Activity的task中新建。


## B的启动模式：singleTop（这个模式启动不会去考虑taskaffinity）
- 现象：
  在当前task即A所在的Task T1 的栈顶做判断如果是B则复用这个B，如果不是B则新加入一个B。
- 记忆要点：
  当前task的栈顶是不是待启动的Activity 如果是复用，如果不是new一个Activity。


## B的启动模式：singleTask（唯一性 唯一性的标注是TaskAffinity）
- 现象：
  B会被启动到B应该所在的task。
- 记忆要点：
  - 会保证Activity会在固定的task中创建。
  - 保证这个task中只有一个目标task。如果存在，会将b之上的Activity出栈。
  - allowtaskreparenting：如果设置了这属性，B会被A启动放置在T1中。
  新启动的Activity放到自己所在的task

## singleinstance（唯一独占性）
逻辑和 singletask相同，不同在于独占一个task，也不允许别的Activity在他之上。

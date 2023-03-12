---
author: Bokmark Ma
title: Compose 入门
date: 2021-10-20T23:35:29+08:00
slug: compose/intro
description: Compose 系列第一期
draft: true
categories:
    - Compose
tags:
    - compose
---

# Compose 启程

> [官方网站](https://developer.android.com/jetpack/compose?hl=zh-cn) 

> [入门文档](https://developer.android.com/jetpack/compose/documentation?hl=zh-cn)

## 如何在现有的项目中启用compose

- 检查你的Android Studio 环境
    - 下载安装 [Arctic Fox](https://developer.android.com/studio?hl=zh-cn#downloads) 版本
    - Android Gradle 插件: 7.0.0 以上
    - Kotlin 版本: 	1.7.10 以上 【必须与compose compailer 版本对应】
    - kotlin 版本必须和compose版本一一对应。查询 https://developer.android.com/jetpack/androidx/releases/compose-kotlin
       
- 在相应的module 中添加
    ```kotlin
    android {
    defaultConfig {
            ...
            minSdkVersion 21
        }

        buildFeatures {
            // Enables Jetpack Compose for this module
            compose true
        }
        ...

        // Set both the Java and Kotlin compilers to target Java 8.
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        kotlinOptions {
            jvmTarget = "1.8"
        }

        composeOptions {
            // 这个compiler 的版本必须和 kotlin的版本一一对应
            kotlinCompilerExtensionVersion "1.3.0"
        }
    }
    ```


## 在过程中遇到的问题

- 关于 compose 和hilt 在现有项目的疑似有冲突的解决方案。或者说在现有项目中引入compose的方法。
    首先，compose 和 hilt 不存在冲突。其次，当你需要引入compose的时候需要修改很多的配置，这时候这些配置会导致你的项目编译不过，从而导致了你的冲突。所以，解决方案如下
    - 第一步，修改你的 android gradle plugin 版本
        `androidGradlePlugin = "com.android.tools.build:gradle:7.0.3"`
    - 第二步，修改你的kotlin版本，包括 kotlin的gradle 插件版本 到 `1.5.31`
    - 第三步，修改你的 compileSdkVersion 和 targetSdkVersion 到 `31`
    - 第四步，修改你的 `androidx.activity:activity-ktx:1.3.1`
    - 第五步，如果有 buildToolsVersion 修改到`30.0.2`
    - 第六步，如果你使用了hilt，请升级到`2.39.1`
       > ```
       > com.google.dagger:hilt-compiler:$version
       > com.google.dagger:hilt-android-compiler:$version
       > ```
       > 是同一个东西，使用`com.google.dagger:hilt-compiler:$version` 即可
    - 第七步，这时候你重新编译，你应该会遇到 包括但不限于 以下问题：
        - kotlin语法问题，比如可空类型的编译报错
        - hilt 的 ApplicationComponent 作用域移除，该怎么修复自行google
        - android manifest 的activity 节点必须添加 `android:exported="false"`
    - 第八步，添加compose的配置，如何添加，参看上文。

## compose 大部分依赖奉上
```kotlin
const val version = "1.0.4"

const val foundation = "androidx.compose.foundation:foundation:${version}"
const val layout = "androidx.compose.foundation:foundation-layout:${version}"
const val ui = "androidx.compose.ui:ui:${version}"
const val uiUtil = "androidx.compose.ui:ui-util:${version}"
const val runtime = "androidx.compose.runtime:runtime:${version}"
const val material = "androidx.compose.material:material:${version}"
const val animation = "androidx.compose.animation:animation:${version}"
const val tooling = "androidx.compose.ui:ui-tooling:${version}"
const val iconsExtended = "androidx.compose.material:material-icons-extended:$version"
const val uiTest = "androidx.compose.ui:ui-test-junit4:$version"
const val activityCompose = "androidx.activity:activity-compose:1.3.1"
const val viewModelCompose = "androidx.lifecycle:lifecycle-viewmodel-compose:2.4.0-rc01"
const val navigationCompose = "androidx.navigation:navigation-compose:2.4.0-alpha10"
const val constraintLayoutCompose = "androidx.constraintlayout:constraintlayout-compose:1.0.0-rc01"
```
---
author: Bokmark Ma
title: Android Gradle Implementation BoM
date: 2022-04-19
description: 多个构件同时依赖可以使用 BoM 来进行版本控制
slug: o/BoM
categories:
    - Other
tags:
    - gradle
    - dependency
    - compose
---

# Android Gradle 多个依赖版本控制

> 故事从compose开始说起 [compose-bom](https://developer.android.com/jetpack/compose/bom/bom?hl=zh-cn)
```grovvy
dependencies {
    def composeBom = platform('androidx.compose:compose-bom:2022.12.00')
    implementation composeBom
    androidTestImplementation composeBom
    // 添加 compose的依赖
    implementation "androidx.compose.ui:ui"
    implementation "androidx.compose.ui:ui-tooling-preview"
    implementation 'androidx.compose.material:material'
    // 添加 ui test 的依赖
    androidTestImplementation "androidx.compose.ui:ui-test-junit4"
    debugImplementation "androidx.compose.ui:ui-tooling:"
    debugImplementation "androidx.compose.ui:ui-test-manifest"
}
```

## [BOM(Bill of Material)](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)

当一系列的依赖各有各的版本号时，想要找到对应的最新稳定版不是一件容易的事情，特别是各个版本之间可能存在细微的差异从而导致意料之外的错误发生。  
这时候就需要 bom 的挺身而出。  

## compose bom
借助 Compose 物料清单 (BoM)，您只需指定 BoM 的版本，即可管理所有 Compose 库版本。BoM 本身包含指向不同 Compose 库的稳定版的链接，以便它们能够很好地协同工作。在应用中使用 BoM 时，您无需向 Compose 库依赖项本身添加任何版本。更新 BoM 版本时，您使用的所有库都会自动更新到新版本。  

如需了解哪些 Compose 库版本已映射到特定 [BoM 版本，请查看 BoM 到库的版本映射。](https://developer.android.com/jetpack/compose/bom/bom-mapping?hl=zh-cn)

### 为什么 BoM 中不包含 Compose 编译器库？
Compose Kotlin 编译器扩展 (androidx.compose.compiler) 未关联到 Compose 库版本。相反，它会关联到 Kotlin 编译器插件的版本，并与 Compose 的其余部分分开发布，因此请务必使用与您的 Kotlin 版本兼容的版本。您可以在 [Compose 与 Kotlin 的兼容性对应关系](https://developer.android.com/jetpack/androidx/releases/compose-kotlin?hl=zh-cn)页面找到映射到每个插件版本的 Kotlin 版本。

### 如何使用 BoM 中未指定的库版本？
如下。如果指定了特殊的版本，可能会导致相关依赖的版本都进行修改。
```grovvy
dependencies {
    // Import the Compose BOM
    implementation platform('androidx.compose:compose-bom:2023.01.00')

    // Override Material Design 3 library version with a pre-release version
    implementation 'androidx.compose.material3:material3:1.1.0-alpha01'

    // Import other Compose libraries without version numbers
    // ..
    implementation 'androidx.compose.foundation:foundation'
}
```

### BoM 只会管理版本号，并不会添加所有的compose 库
要在您的应用中实际添加和使用 Compose 库，您必须在模块（应用级）Gradle 文件（通常是 app/build.gradle）中将每个库声明为单独的依赖项行。  
使用 BoM 可确保应用中的任何 Compose 库版本兼容，但 BoM 实际上并不会将这些 Compose 库添加到您的应用中。

### 为什么建议使用 BoM 管理 Compose 库版本
今后，Compose 库将单独进行版本控制，这意味着版本号将开始按照自己的节奏递增。每个库的最新稳定版本已经过测试，并保证能够很好地协同工作。不过，找到每个库的最新稳定版本可能比较困难，而 BoM 会帮助您自动使用这些最新版本。

### 我是否必须使用 BoM？
不会。您仍然可以选择手动添加各个依赖项版本。不过，我们建议您使用 BoM，因为这样您可以更轻松地同时使用所有最新的稳定版本。


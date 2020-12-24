---
title: "Gradle connect Reset"
date: 2020-11-14T12:20:31+08:00
draft: false
summary: "Could not install Gradle distribution from 'https://services.gradle.org/distributions/gradle-6.5.1-all.zip'."
---

> Could not install Gradle distribution from 'https://services.gradle.org/distributions/gradle-6.5.1-all.zip'.

换了电脑之后发现gradle 下载不下来了，在研究了一番之后我找到了一下的方法。



- 到这个页面https://services.gradle.org/distributions/ 下载你需要的内容，可以用迅雷等工具下载。记得下载all.zip 的包

- cd 到  {user}/.gradle/wrapper/dist 目录之下

- android studio 中sync 一下，将会创建对应的目录

- 这个是项目目录下的gradle-wrapper.properties内容

  ```properties
  distributionBase=GRADLE_USER_HOME
  distributionPath=wrapper/dists
  zipStoreBase=GRADLE_USER_HOME
  zipStorePath=wrapper/dists
  distributionUrl=https\://services.gradle.org/distributions/gradle-6.5.1-all.zip
  ```

  比如上面这个将会在  {user}/.gradle/wrapper/dist 目录下创建

  `gradle-6.5.1-all/cdund22i8guosqylfo49op4dv`  类似于这样的一个目录

- cd 进去

  ```
  drwxr-xr-x  10 bokmark  staff        320 11 18 21:59 gradle-6.5.1
  -rwxr-xr-x@  1 bokmark  staff  145830284 11 18 21:59 gradle-6.5.1-all.zip
  -rw-r--r--   1 bokmark  staff          0 11 18 21:59 gradle-6.5.1-all.zip.lck
  -rw-r--r--   1 bokmark  staff          0 11 18 21:59 gradle-6.5.1-all.zip.ok
  ```

  这是成功的目录结构。

  ```
  -rw-r--r--  1 bokmark  staff        0 11 19 10:11 gradle-6.1.1-all.zip.lck
  -rw-r--r--  1 bokmark  staff  1156054 11 19 10:11 gradle-6.1.1-all.zip.part
  ```

  这是失败的目录结构。

- so，我们如果遇到 gradle 无法sync  的问题，就是先找到这个目录`gradle-6.5.1-all/cdund22i8guosqylfo49op4dv`

- 然后 `rm -rf *`

- 然后将通过迅雷下载好的 https://services.gradle.org/distributions/  这个里面下载的`gradle-xxx-all.zip` 放到这个目录下

- android studio中 sync 一下。

- 成功sync

- 如果需要别的版本的gradle可以再如法炮制

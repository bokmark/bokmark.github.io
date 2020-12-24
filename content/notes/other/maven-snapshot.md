---
title: "Maven snapshot"
date: 2020-11-14T12:20:31+08:00
draft: false
summary: "Maven 快速开发迭代 版本好 snapshot"
---
> 转载自 https://www.runoob.com/maven/maven-snapshots.html

## Maven 快照(SNAPSHOT)

一个大型的软件应用通常包含多个模块，并且通常的场景是多个团队开发同一应用的不同模块。举个例子，设想一个团队开发应用的前端，项目为 app-ui(app-ui.jar:1.0)，而另一个团队开发应用的后台，使用的项目是 data-service(data-service.jar:1.0)。

现在可能出现的情况是开发 data-service 的团队正在进行快节奏的 bug 修复或者项目改进，并且他们几乎每隔一天就要发布库到远程仓库。 现在如果 data-service 团队每隔一天上传一个新版本，那么将会出现下面的问题：

- data-service 团队每次发布更新的代码时都要告知 app-ui 团队。
- app-ui 团队需要经常地更新他们 pom.xml 文件到最新版本。

为了解决这种情况，快照的概念派上了用场。
___
### 什么是快照?

快照是一种特殊的版本，指定了某个当前的开发进度的副本。不同于常规的版本，Maven 每次构建都会在远程仓库中检查新的快照。 现在 data-service 团队会每次发布更新代码的快照到仓库中，比如说 data-service:1.0-SNAPSHOT 来替代旧的快照 jar 包。
___
### 项目快照 vs 版本

对于版本，如果 Maven 以前下载过指定的版本文件，比如说 data-service:1.0，Maven 将不会再从仓库下载新的可用的 1.0 文件。若要下载更新的代码，data-service 的版本需要升到1.1。

快照的情况下，每次 app-ui 团队构建他们的项目时，Maven 将自动获取最新的快照(data-service:1.0-SNAPSHOT)。
#### app-ui 项目的 pom.xml 文件

app-ui 项目使用的是 data-service 项目的 1.0 快照。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>app-ui</groupId>
   <artifactId>app-ui</artifactId>
   <version>1.0</version>
   <packaging>jar</packaging>
   <name>health</name>
   <url>http://maven.apache.org</url>
   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
   <dependencies>
      <dependency>
      <groupId>data-service</groupId>
         <artifactId>data-service</artifactId>
         <version>1.0-SNAPSHOT</version>
         <scope>test</scope>
      </dependency>
   </dependencies>
</project>
```
#### data-service 项目的 pom.xml 文件

data-service 项目为每次小的改动发布 1.0 快照。
```xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>data-service</groupId>
   <artifactId>data-service</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>jar</packaging>
   <name>health</name>
   <url>http://maven.apache.org</url>
   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
</project>

```
虽然，快照的情况下，Maven 在日常工作中会自动获取最新的快照， 你也可以在任何 maven 命令中使用 -U 参数强制 maven 现在最新的快照构建。

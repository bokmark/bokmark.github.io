---
title: '创建并上传mavenSdk'
date: '2021-02-01'
weight: 2
summary: "创建并上传mavenSdk"
---

# maven 上传 sdk的正确打开方式

## 概念明确并区分
- **公司级自建maven**:公司内部创建的maven（具体内容询问公司的开发）
- **mavenLocal**:本地电脑中创建的本地仓库（{user}/.m2/repository/）
- **jcenter、google、maven等**: 外部开源的仓库，大部分的依赖都是从这些外部开源的仓库中下载的。
```groovy
repositories {
    maven {url 'https://maven.aliyun.com/repository/public/'}
    mavenLocal()
    google()
    jcenter()
    maven {url 'http://{company.ip}/nexus/content/repositories/thirdparty/'}
    maven {url 'http://{company.ip}/nexus/content/repositories/snapshots/'}
}
```
一般的maven仓库如果使用nexus的方式搭建会有如下几个重要的仓库
- snapshots （快照库）
- releases （release库）
- thirdparty（第三方库）
- public（公开库）

## pom 文件
[这是一个retrofit的pom文件](https://repo1.maven.org/maven2/com/squareup/retrofit2/retrofit/2.9.0/retrofit-2.9.0.pom)

从这里我们可以看到几个关键的点
```xml
<groupId>com.squareup.retrofit2</groupId>
<artifactId>retrofit</artifactId>
<version>2.9.0</version>
<dependencies>
  <dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.14.9</version>
    <scope>compile</scope>
  </dependency>
</dependencies>
```
- 一个maven的依赖可以定义他需要的依赖
- 相应的groupId artifactId version 等等信息都是定义在pom文件中的。

**可以说 pom文件就是maven的灵魂**

## Android中的aar
我们假设当前的module为AModule，他依赖于BModule。
当我们运行AModule的assemble task之后，将会在build目录得到一个aar。
这个aar中只包括这些内容
- AModule中包含的所有java、kotlin代码
- AModule中的aidl文件
- AModule中的res资源文件
- AModule中自动生成的那些java或者class 代码

这些内容中不包含BModule的代码。<br>
**所以这就意味着如果我们只能上传这个AModule中的代码，AModule所依赖的BModule的代码是无法编译进AModule的aar中的，因为会有一个BModule的aar存在**

所以我们需要做的是尽量将sdk的代码合并到一个module。**那么确实无法做到这一点该怎么办呢？** 这里介绍两种方法：
- [fat-aar](https://github.com/kezong/fat-aar-android)。一个gradle 插件将依赖module的代码强行合并进本module。（但是不推荐）
- 将BModule的aar 也上传到仓库中。这样我们就可以通过引入依赖的依赖的方式来将BModule引入。

我们接下来的讨论的就是（将BModule的aar 也上传到仓库中）的这种方式。

## 如何将一个module的aar 上传到maven

### 配置maven账号
项目的local.properties添加
```
maven.username=xxx
maven.password=xxx
```

### 添加代码
- 添加plugin `maven-publish`
- 添加上传相关task

```groovy

Properties properties = new Properties()
def localPropertiesFile = new File('../local.properties')
if (null != localPropertiesFile && localPropertiesFile.exists()) {
  properties.load(localPropertiesFile.newDataInputStream())
}

afterEvaluate {
    publishing {
        publications {
            // Creates a Maven publication called "release".
            release(MavenPublication) {
                // Applies the component for the release build variant.
                from components.release
                artifact androidJavadocsJar
                artifact androidSourcesJar
                pom {
                    name = "{maven name}"
                    description = "{this is a maven}"
                    developers { // 这个模块可以缺省
                        developer {
                            id = "{id}"
                            name = "{developer.name}"
                            email = "{developer.name}"
                        }
                    }
                }
                // You can then customize attributes of the publication as shown below.
                // implementation "groupId:artifactId:version"
                groupId = "{groupId}"
                artifactId = "{artifact}"
                version = "{version}"
            }
        }
        repositories {
            maven {
                url "maven url" //maybe like http://{ip}/nexus/content/repositories/snapshots/
                credentials {
                    username propterties.getProperty("maven.username")
                    password propterties.getProperty("maven.password")
                }
            }
        }
    }
}

task androidJavadocs(type: Javadoc) {
    failOnError false
    options.addStringOption('Xdoclint:none', '-quiet')
    options.addStringOption('encoding', 'UTF-8')
    options.addStringOption('charSet', 'UTF-8')
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    android.libraryVariants.all { variant ->
        if (variant.name == 'release') {
            owner.classpath += variant.javaCompileProvider.get().classpath
        }
    }
    exclude '**/R.html', '**/R.*.html', '**/index.html'
}

task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
    archiveClassifier.set('javadoc')
    from androidJavadocs.destinationDir
}

task androidSourcesJar(type: Jar) {
    archiveClassifier.set('sources')
    from android.sourceSets.main.java.srcDirs
}
```
**当然相关信息你都可以独立在一个gralde文件中进行管理**

#### **当你配置好这些的时候你gradle sync 一下 会在你的gradle window中看到publishing 类别**
其中有这些task
- generateMetadataFileForReleasePublication
- generatePomFileForReleasePublication
- publish
- publishAllPublicationsToMavenRepository
- publishReleasePublicationToMavenLocal
- publishReleasePublicationToMavenRepository
- publishToMavenLocal

点击publishReleasePublicationToMavenRepository 就可以上传到maven中了
**不过在上传之前你最好在本地验证一下**

## 依赖的依赖
这种方式会自动的将你的依赖添加到 pom文件中，所以你只需要在需要上传的module中添加上面的代码即可。这样就可以循环上传所有的sdk


## SNAPSHOT
```groovy
repositories {
    maven {
        url "maven url" //maybe like http://{ip}/nexus/content/repositories/snapshots/
        credentials {
            username propterties.getProperty("maven.username")
            password propterties.getProperty("maven.password")
        }
    }
}
```
在这里定义的url中写入的snapshots的仓库地址 然后定义
```groovy
version = "{version}"
```
的时候加上`-SNAPSHOT`的后缀就可以将sdk上传到快照仓库。

### 修改snapshot的依赖的默认拉取时长
默认情况下，maven针对snapshot的依赖会每隔24小时强行刷新一次，而我们可以通过配置啦修改这个默认的刷新时长
添加在 module的build.gradle 的**根节点**上
```groovy
configurations.all {
    // check for updates every build
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
```

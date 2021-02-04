---
title: 'Java IO 的基础应用'
date: 2018-11-28T15:14:39+10:00
weight: 4
summary: "java io 的基础应用"
---

# Java中IO基础应用

> [课外阅读 okio](https://github.com/square/okio)

## Input\\Output 方向理解

从内存的角度来理解input\\output的方向。
- 内存输出到外部 output **Write**
- 内存接受输入 input **Read**
{{<mermaid>}}
graph TD;
    内存-- output -->外部;
    外部-- input -->内存;
{{</mermaid>}}


## 字节流
InputStream\\OutputStream

### 举个栗子
```Kotlin
val inputStream = DataInputStream(BufferedInputStream(FileInputStream(File("/sdcard/a.png"))))
```

### 几个主要实现类的类图（以InputStream为例）

先看几个常见的Stream类
- java.io.BufferedInputStream
- java.io.ByteArrayInputStream
- java.io.DataInputStream
- java.io.FilterInputStream
我们来分析他的继承关系

{{<mermaid>}}
classDiagram
    InputStream <|-- FileInputStream
    InputStream <|-- ObjectInputStream
    InputStream <|-- FilterInputStream
    FilterInputStream <|-- DataInputStream
    FilterInputStream <|-- BufferedInputStream
    InputStream <|-- ByteArrayInputStream

{{</mermaid>}}

### 几个主要stream的功能介绍
- BufferedInputStream
- FileInputStream
- DataInputStream
- ByteArrayInputStream
- ObjectInputStream

## 字符流
Reader\\Writer

### 举个栗子
```Kotlin
val inputStream = FileInputStream(File("/sdcard/a.png"))
val reader = BufferedReader(InputStreamReader(inputStream, "GBK"))
```

### 几个主要实现类的类图（以Reader为例）

先看几个常见的Reader类
- CharArrayReader
- BufferedReader
- FilterReader
- PipedReader
我们来分析他的继承关系

{{<mermaid>}}
classDiagram
    Reader <|-- BufferedReader
    Reader <|-- LineNumberReader
    Reader <|-- InputStreamReader
    Reader <|-- CharArrayReader
    InputStreamReader <|-- FileReader
    Reader <|-- FilterReader
    Reader <|-- StringReader
    FilterReader <|-- PushbackReader

{{</mermaid>}}

## RandomAccessFile

## FileChannel(nio)

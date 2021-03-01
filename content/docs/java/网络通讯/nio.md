---
title: 'BIO,NIO,AIO'
date: 2020-12-01T15:14:39+10:00
weight: 1
tags: ["tcp、ip"]
categories: ["网络协议"]
---

# 网络通信基本常识
> [课外阅读-bio 与nio的区别](https://tech.meituan.com/2016/11/04/nio.html)
> [课外阅读-bio 与nio的区别2](https://blog.csdn.net/forezp/article/details/88414741)
> [课外阅读-bio 与nio的区别2](http://tutorials.jenkov.com/java-nio/nio-vs-io.html)

## 基础知识
- socket
- 短连接
- 长

### 11
读写是否需要双开线程

## nio
io 多路复用，面向缓冲区

### 什么是nio
- io 多路复用
- channel
  - serversocketchannel
  - socketchannel
- selector
- selectionkey
  - op_accept
  - op_connect
  - op_read
  - op_write
    - 为什么op_write 事件很少注册 （只有极少数才会注册写事件，因为写事件在发送buffer有空闲时就会触发）
- socket
- buffer

### bytebuffer
- 客户端传输数据和接收数据需要用两个bytebuffer  
- 直接内存
- 推上分配

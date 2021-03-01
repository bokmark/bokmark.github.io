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

## 三种io

- bio
- nio
- AIO

## bio
面向流的io方式

### 什么是bio

### 举个栗子：BIO
```Java
ExecutorService executor = Excutors.newFixedThreadPollExecutor(100);//线程池

ServerSocket serverSocket = new ServerSocket();
serverSocket.bind(8088);
while(!Thread.currentThread.isInturrupted()){//主线程死循环等待新连接到来
   Socket socket = serverSocket.accept();
   executor.submit(new ConnectIOnHandler(socket));//为新的连接创建新的线程
}

class ConnectIOnHandler extends Thread{
    private Socket socket;
    public ConnectIOnHandler(Socket socket){
       this.socket = socket;
    }
    public void run(){
      while(!Thread.currentThread.isInturrupted()&&!socket.isClosed()){死循环处理读写事件
          String someThing = socket.read()....//读取数据
          if(someThing!=null){
             ......//处理数据
             socket.write()....//写数据
          }
      }
    }
}

```
相较于 nio 优点在于交互数据的效率更高
缺点就是在处理多路流的时候由于一个线程处理一个输入从而导致多路流的涉及到多线程处理。多线程又是一个消耗巨大的内容，特别在jvm的模型中 java的线程与进程相似。

---
author: Bokmark Ma
title: "Install Go on linux"
date: 2021-10-29T17:20:58+08:00 
description: 如何在一个全新的云主机搭建go的服务端
categories:
    - Go
tags:
    - go
    - flutter
---

# 在腾讯云的服务器搭建go的服务端

> 腾讯云服务器的后台 https://console.cloud.tencent.com/lighthouse/instance/index
> 双11 50元一年买的最入门版的服务器。


## 安装开发环境

### go 的环境安装

ubuntu 上一键搞定
```
sudo apt-get install golang

go version
```

如果能出现go的版本号 `go version go1.13.8 linux/amd64` 就成功了。

### mysql的安装
> https://linuxize.com/post/how-to-install-mysql-on-ubuntu-18-04/

```
sudo apt update
sudo apt install mysql-server
```
如果安装完成，mysql将会自动启动，我们可以通过 
```
sudo systemctl status mysql
```
来查看mysql 是否启动了。如果启动成功你会看到类似的一个echo。

```
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-10-29 17:30:24 CST; 20s ago
   Main PID: 8885 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 2274)
     Memory: 353.2M
     CGroup: /system.slice/mysql.service
             └─8885 /usr/sbin/mysqld
```

### go的网络框架选择：Beego

> [框架](https://beego.me/)

```
go get github.com/beego/beego/v2@v2.0.0
```

创建hello.go
```go
package main

import "github.com/beego/beego/v2/server/web"

func main() {
    web.Run()
}
```

编译运行

```
go build -o hello hello.go
./hello
```

打开浏览器并访问[http://localhost:8080](http://localhost:8080)

[可以点击开发文档查看](https://beego.me/docs)


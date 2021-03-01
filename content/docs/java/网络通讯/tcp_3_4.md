---
title: 'TCP 三次握手、四次挥手'
date: 2020-12-01T15:14:39+10:00
weight: 1
tags: ["tcp、ip"]
categories: ["网络协议"]
---
# TCP 的三次握手、四次挥手

## 三次握手

由于tcp 是有具有数据排序超时重传的功能，所以需要交换服务器和客户端的序列号的起始值。<br>  
**SYN**：Synchronize Sequence Numbers <br>
**ACK**：Acknowledge character
{{<mermaid>}}
sequenceDiagram
    Note left of Client: SYN_SENT
    activate Client
    Client->>Server: SYN=1, seq = 12345（请求建立连接）
    deactivate Client
    activate Server
    Note right of Server: SYN_RCVD
    Server->>Client: SYN=1,ACK=1,ack=12346,seq=45678(响应请求连接请求，并发送服务器的连接请求)
    deactivate Server
    activate Client
    Note left of Client: ESTABLISHED
    Client->>Server: ACK=1,ack=45679(收到服务器发送的连接请求，并响应)
    deactivate Client
    activate Server
    Note right of Server: ESTABLISHED
    deactivate Server
{{</mermaid>}}

### 为什么是三次握手，而不是四次握手，不是二次握手
{{< hint info >}}
这是有效建立连接必需步骤：双端需要交换序列号值起始值 的所需要的最小次数。
{{</hint>}}

### 三次握手的漏洞

{{< hint info >}}
- syn 宏泛攻击（ddos的一种）
  虚假的发起syn的请求，（第一次握手），服务器返回第二次握手，并且等待。这将会大量消耗服务器资源。（超时的时间一般都是分钟级别的）
- syn 宏泛攻击 解决方案
  - 无效连接监控释放
    新起一个线程轮询服务器的连接，将无效的连接释放。
  - 延缓tcb分配方法
  - 防火墙
    组织无效的请求发送到服务器
{{</hint>}}

### 课外阅读
- 网络嗅探
- 原始套接字（直接抓起网卡中的数据，越过网络层传输层）
- tcpdump
- websocket：应用层协议（《HTML5 WebSocket权威指南》）

## 四次挥手
> - 第一次挥手：客户端发送关闭请求
> - 第二次挥手：服务端响应客户端关闭请求
> - 第三次挥手：服务端关闭请求
> - 第四次挥手：客户端发送关闭确认请求

{{< mermaid >}}
sequenceDiagram
    Note left of 主动关闭方: FIN_WAIT_1(close主动关闭)
    Note right of 被动关闭方: CLOSE_WAIT
    activate 主动关闭方
    主动关闭方->>被动关闭方: FIN=1, seq = 12345（请求关闭连接）
    deactivate 主动关闭方
    activate 被动关闭方
    被动关闭方->>主动关闭方: ACK=1,seq=12346(确认收到请求，并发送服务器的连接请求)
    deactivate 被动关闭方
    activate 主动关闭方
    Note left of 主动关闭方: FIN_WAIT_2
    被动关闭方->>主动关闭方  : FIN=1,seq=45678(确认收到请求，并发送服务器的连接请求)
    activate 被动关闭方
    Note right of 被动关闭方: CLOSE
    deactivate 主动关闭方
    deactivate 被动关闭方
    activate 主动关闭方
    主动关闭方->>被动关闭方: ACK=1, seq = 45679（请求关闭连接）
    activate 被动关闭方
    Note left of 主动关闭方: TIME_WAITING（2*MSL）
    deactivate 主动关闭方
    Note right of 被动关闭方: CLOSED
    deactivate 被动关闭方
{{< /mermaid >}}

- MSL: 最长报文段的寿命。一个ip报文数据在网络中能存活的最长时间。（RFC的协议定义中，是两分钟。常规定义时间是30秒）（1分钟~4分钟）
### 为什么是四次挥手
{{< hint info >}}
这是因为tcp是全双工的，所以连接双方都需要和对方明确不会再有数据发送
{{</hint>}}
### 为什么第2步和第3步 不合并发送
{{< hint info >}}
可以合并发送，这不是硬性要求，这是rfc文档定义的规则，当被动关闭方收到fin的报文，并且自己也没有数据要发送的时候，就可以将第二步和第三步合并发送。但是如果被动关闭方还有别的数据需要发送，那么就需要将两个步骤分开
{{</hint>}}
### 为什么主动发送关闭端 需要一个TIME_WAITING 状态
{{< hint info >}}
- 确保可以对被动关闭方发送的fin报文进行响应：预防fin报文的ack报文丢失，从而导致服务器重发了fin报文，但是客户端无法ack。
- 确保服务器延迟发送的数据不会错误的发送到别的端口上：因为存在端口冲突，当主动发送方关闭了tcp连接的时候端口号可能被别的程序得到，从而导致被动关闭方延迟发送或者还在网络中的数据包被错误的发送的新开的程序端口上。
{{</hint>}}

### keepalive 协议中定义了75分钟
需要应用层定义心跳机制

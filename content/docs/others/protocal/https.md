---
title: 'HTTPS 与http的区别'
date: 2021-02-28
weight: 1
---
# HTTPS

## http 协议的缺点
- http协议是明文传输
- 没有对内容的完整性进行校验

## HTTPS 在http 和tcp之间加入一层 ssl。
{{<mermaid>}}
sequenceDiagram
    participant C as Client
    participant S as Server
    participant CA as CA
    C->>S: 发起ssl握手请求。附带支持的密码套件。
    S->>C: 收到请求。告知采用的密码套件。
    S->>C: 返回一个CA证书中的公钥。RSA非对称加密。
    C->>C: 先在本地查找服务器证书的根证书。
    C->>C: 利用根证书的公钥（如果不是受信的签发机构，找到签发机构的数字证书的签发机构是不是在列表中，一直循环。）解密服务器证书，得到摘要数据，与明面上的摘要信息比较，如果相同则通过。
    C->>S: 使用服务器证书的公钥加密 一个随机值（由前面协商的算法决定的）密钥。并发送给服务器
    S->>C: 服务器用自己的私钥解密，利用这个随机值开始通信。
{{</mermaid>}}

- 为什么要有CA
  预防中间人攻击。
- 为什么要用对称加密。
  因为非对称算法的消耗巨大。对称加密的性能相较于很大。

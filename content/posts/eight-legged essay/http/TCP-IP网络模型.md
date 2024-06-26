---
title: 'TCP/IP网络模型'
date: 2022-09-22T17:22:23+08:00
tags: [http]
draft: true
---

TCP/IP 模型是互联网的基础，它是一系列网络协议的总称。这些协议可以划分为四层，如下：

- 应用层，针对特定应用的协议，电子邮件，文件传输等，通俗点说就是让用户程序能访问网络，并获得各种网络服务（http、FTP、DNS...）
- 传输层，根本任务是提供端到端的可靠通信（TCP、UDP）
- 网络层，地址管理与路由选择（IP...）
- 链路层，互连设备之间传送和识别数据帧。

![](https://cdn.jsdelivr.net/gh/yokiizx/picgo@main/img/202210202335427.png)

---

作为前端，在应用层主要关注 http 协议，在 [http](https://yokiizx.site/posts/http/http%E5%89%8D%E4%B8%96%E4%BB%8A%E7%94%9F/) 中已经有了详细介绍。

在传输层，核心的就是 TCP 了，三次握手四次挥手那更是老生常谈的问题了。简单记录下常见面试题，详细的见文末参考文章，或者自行 google~

![](https://cdn.jsdelivr.net/gh/yokiizx/picgo@main/img/202210210008339.png)

1. 为啥握手是三次  
   双端确认各自及对方的接收、发送能力正常。
   - 第一次，服务端确认了客户端的发送能力和服务端的接收能力正常
   - 第二次，客户端确认了服务端的接收、发送能力,客户端的接收、发送能力正常
   - 第三次，服务端确认了服务端的发送能力和客户端的接收能力正常  
     如果没有第三次，一旦第二次出了问题，客户端会发起第二个请求，而丢失的报文段只是某些网络结点长时间滞留了，延误到连接释放以后的某个时间才到达服务端，这第二个迟来的连接会被客户端忽略，服务端一直等待客户端发送数据，浪费资源。
2. 三次握手中可以携带数据吗  
   前两次不行，第三次可以。前两次还没建立连接，而且第一次如果能携带数据会被利用，在 SYN 携带大量数据来攻击服务器。
3. SYN 攻击是什么  
   服务器端的资源分配是在二次握手时分配的。就是利用大量不存在的 IP 来向服务器发送连接请求，服务端回复后等待第三次连接，因为客户端 IP 根本不存在，所以会成为半连接，服务端会不断重发直至超时，大量的半连接导致正常的 SYN 请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。SYN 攻击是一种典型的 DoS/DDoS 攻击。

![](https://cdn.jsdelivr.net/gh/yokiizx/picgo@main/img/202210210009002.png)

1. 为啥挥手是四次  
   因为服务端接收到 FIN 的时候，很可能并不能立刻关闭连接，所以先只回复了应答 ACK 报文，告诉服务端我知道了，等服务端报文发送完毕后，再发送 FIN 报文同步关闭。
2. 四次会后释放连接，等待 2MSL 的意义是什么 MSL（Maximum Segment Lifetime）
   1. 说白了就是让客户端最后的一个确认报文 ACK 能够到达服务器。  
      因为网络是不稳定的，有可能丢失，等待 2MSL 就是保证服务端没有收到 ACK 后重新发起一次关闭请求，客户端也刷新等待时间从头计时；如果不等待直接关闭，一旦丢失，服务端将不能正确关闭。
   2. 防止“已失效的连接请求报文段”出现在本连接中  
      客户端在发送完最后一个 ACK 报文段后，再经过 2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失，使下一个新的连接中不会出现这种旧的连接请求报文段。

---

计算机基础

- 位（bit）：二进制单位，一个 0 或一个 1
- 字节（Byte）：8 bit = 1 Byte，最小存储单位。 这就是为什么百兆宽带下行速度往往只有 15 MB/s 左右

## 参考

- [面试官，不要再问我三次握手和四次挥手](https://mp.weixin.qq.com/s/WI9045Sd7gRsE-WZ5x8tcA)

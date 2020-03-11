---
title: http1.0和http1.1以及http2.0区别
date: 2020-03-11 16:17:22
categories: [Network]
tags:
---

## http1.0与http1.1的区别

1)缓存处理，在http1.0 Expires,Last-Modified/If-Modified-Since，HTTP1.1增加Cache-Control、Etag、If-None-Match等更多可供选择的缓存头来控制缓存策略。

2)带宽优化及网络连接的使用，http1.0存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，http1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

3)错误通知的管理，在http1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

4)Host头处理，在http1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

通用头域:

- Cache-Control: 缓存头域 => 常见值为no-cache(不允许缓存), no-store(无论请求还是响应均不允许缓存), max-age(规定可以客户端可以接受多长生命期的数据)
- Keep-Alive: 使得服务端和客户端的链接长时间有效
- Date: 信息发送的时间
- Host: 请求资源的主机IP和端口号
- Range: 请求资源的某一部分
- User-Agent: 发出请求的用户的信息(鉴权)

5)长连接，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

## http2.0和http1.x的区别

1）http1的解析是基于文本协议的格式解析,而http2.0的协议解析是二进制格式,更加的强大

2）多路复用(Mutiplexing) : 一个连接上可以有多个request,且可以随机的混在一起,每个不同的request都有对应的id,服务端可以通过request_id来辨别,大大加快了传输速率

3）header压缩: http1.x中的header需要携带大量信息.而且每次都要重复发送.http2.0使用encode来减少传输的header大小.而且客户端和服务端可以各自缓存(cache)一份header filed表,避免了header的重复传输,还可以减少传输的大小.

4）服务端推送(server push): 可以通过解析html中的依赖,只能的返回所需的其他文件(css或者js等),而不用再发起一次请求.

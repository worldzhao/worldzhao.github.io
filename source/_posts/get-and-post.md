---
title: HTTP中GET与POST的区别
date: 2017-09-14
tags: [ajax]
categories: http
---

1.  GET 在浏览器回退时是无害的，而 POST 会再次提交请求。
2.  GET 产生的 URL 地址可以被 Bookmark，而 POST 不可以。
3.  GET 请求会被浏览器主动 cache，而 POST 不会，除非手动设置。
4.  GET 请求只能进行 url 编码，而 POST 支持多种编码方式。
5.  GET 请求参数会被完整保留在浏览器历史记录里，而 POST 中的参数不会被保留。
6.  GET 请求在 URL 中传送的参数是有长度限制的，而 POST 没有。
7.  对参数的数据类型，GET 只接受 ASCII 字符，而 POST 没有限制。
8.  GET 比 POST 更不安全，因为参数直接暴露在 URL 上，所以不能用来传递敏感信息。
    GET 参数通过 URL 传递，POST 放在 Request body 中。

GET 和 POST 是什么?

HTTP 协议中的两种发送请求的方法。

HTTP 是什么?

HTTP 是基于 TCP/IP 的关于数据如何在万维网中如何通信的协议。

HTTP 的底层是 TCP/IP。所以 GET 和 POST 的底层也是 TCP/IP，也就是说，GET/POST 都是 TCP 链接。GET 和 POST 能做的事情是一样一样的。

你要给 GET 加上 request body，给 POST 带上 url 参数，技术上是完全行的通的。

业界不成文的规定是，(大多数)浏览器通常都会限制 url 长度在 2K 个字节，而(大多数)服务器最多处理 64K 大小的 url。超过的部分，恕不处理。如果你用 GET 服务，在 request body 偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然 GET 可以带 request body，也不能保证一定能被接收到哦。

GET 和 POST 还有一个重大区别，

简单的说：

GET 产生一个 TCP 数据包;POST 产生两个 TCP 数据包。

长的说：

对于 GET 方式的请求，浏览器会把 http header 和 data 一并发送出去，服务器响应 200(返回数据);

而对于 POST，浏览器先发送 header，服务器响应 100 continue，浏览器再发送 data，服务器响应 200 ok(返回数据)。

也就是说，GET 只需要汽车跑一趟就把货送到了，而 POST 得跑两趟，第一趟，先去和服务器打个招呼“嗨，我等下要送一批货来，你们打开门迎接我”，然后再回头把货送过去。
因为 POST 需要两步，时间上消耗的要多一点，看起来 GET 比 POST 更有效。因此 Yahoo 团队有推荐用 GET 替换 POST 来优化网站性能。但这是一个坑!跳入需谨慎。为什么?

GET 与 POST 都有自己的语义，不能随便混用。
据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的 TCP 在验证数据包完整性上，有非常大的优点。
并不是所有浏览器都会在 POST 中发送两次包，Firefox 就只发送一次。

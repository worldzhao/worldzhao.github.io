---
title: HTTP中的缓存机制
date: 2017-09-14
tags: [缓存]
categories: http
---

![http缓存机制.png](http://upload-images.jianshu.io/upload_images/4869616-57732386f1ec1dea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图：一个缓存 GET 请求的具体流程（源于《http 权威指南》）

总的来说，客户端从服务器请求数据经历如下基本步骤:

1.  检查是否已缓存：如果请求命中本地缓存则从本地缓存中获取一个对应资源的副本；
2.  检查这个资源是否新鲜：是则直接返回到客户端，否则继续向服务器转发请求，进行再验证。
3.  再验证阶段：服务器接收到请求，然后再验证判断资源是否相同，是则返回 304 not modified，未变更。 否则返回新内容和 200 状态码。
4.  客户端更新本地缓存。

## Stage1: 检查缓存

1.  缓存不存在，直接向服务器发出请求，状态码 200
2.  缓存存在，进入 Stage2

## Stage2: 检查资源是否新鲜

关键字：过期检测、Expires 策略、Cache-control 策略（重点）

1.  缓存资源未过期，使用缓存，状态码 200 from cache
2.  缓存资源过期，进入 Stage3

### 过期检测

服务器用 HTTP1.0 中使用 Expires 首部或 HTTP1.1 中的 Cache-Control:max-age 响应首部来指定过期时间。两者同时设置时，在 HTTP1.1 服务器中，Cache-Control 的优先级高于 Expires

### Expires 策略

Expires 是 Web 服务器响应消息头字段，在响应 http 请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。

不过 Expires 是 HTTP1.0 的东西，现在默认浏览器均默认使用 HTTP1.1，所以它的作用基本忽略。

### Cache-control 策略（重点）

Cache-Control 与 Expires 的作用一致，都是指明当前资源的有效期，控制浏览器是否直接从浏览器缓存取数据还是重新发请求到服务器取数据。这个协定取代了以前的 Expires 指令，在 HTTP/1.1 开始支持，且如果同时设置的话，优先级高于 Expires。

[Cache-control 详解-MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)

如果 Expires 或 Cache-Control:max-age 验证未过期，即资源是新鲜的。则直接返回 200 状态码，使用缓存。这里注意缓存命中和访问原始服务器的响应码都是 200，有些代理缓存会在 via 首部附加额外信息，或者使用 200(from cache)。 对于未明确标识的，可以使用 Date 首部的值和当前时间进行比较，如果响应中的日期比较早，客户端通常可以认为这是一条缓存的响应。

![http缓存机制2.png](http://upload-images.jianshu.io/upload_images/4869616-a3444aa33f0392e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

金字塔最下端即缓存资源未过期，使用缓存，状态码 200 from cache。

如果缓存资源过期了进入 Stage3

## Stage3: 服务器再验证

关键字：If-None-Match /Etag、If-Modified-Since／Last-Modified

1.  资源没有变化，直接返回响应 304 和一个空的响应体。
2.  资源发生了变化，返回新的资源 状态码 200

### If-None-Match /Etag

HTTP 协议规格引入 ETag（被请求变量的实体标记），简单点即服务器响应时给请求 URL 标记，并在 HTTP 响应头中将其传送到客户端，类似服务器端返回的格式：

- Etag：web 服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器觉得）。Apache 中，ETag 的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行 Hash 后得到的。

- If-None-Match：当资源过期时（使用 Cache-Control 标识的 max-age），发现资源具有 Etag 声明，则再次向 web 服务器请求时带上头 If-None-Match（Etag 的值）。web 服务器收到请求后发现有头 If-None-Match 则与被请求资源的相应校验串进行比对，决定返回 200 或 304。
  在客户端第一次发出请求后，HttpReponse Header 中包含 Etag

```
Etag:“5d8c72a5edda8d6a:3239″
```

等于告诉 Client 端，你拿到的这个的资源有表示 ID：5d8c72a5edda8d6a:3239

当客户端下一次请求资源过期时，发现资源具有 Etag 声明，浏览器同时发出一个 If-None-Match 报头(Http RequestHeader)此时包头中信息包含上次访问得到的 Etag:“5d8c72a5edda8d6a:3239″标识。

```
If-None-Match:“5d8c72a5edda8d6a:3239“
```

这样，服务器端就会比对 2 者的 Etag。如果匹配，则返回 304(Not Modified) Response。如果不在匹配，则请求一个新的对象。

### If-Modified-Since／Last-Modified

在浏览器第一次请求某一个 URL 时，服务器端的返回状态会是 200 ，内容是你请求的资源，同时有一个 Last-Modified 的属性标记(HttpReponse Header)此文件在服务期端最后被修改的时间，
格式类似这样：

```
Last-Modified:Tue, 24 Feb 2009 08:01:04 GMT
```

客户端第二次请求此 URL 时，首先会判断是否有缓存以及缓存是否过期，如果缓存过期，浏览器会向服务器传送条件 GET 请求，包含 If-Modified-Since 报头(HttpRequest Header)，询问该时间之后文件是否有被修改过：

```
If-Modified-Since:Tue, 24 Feb 2009 08:01:04 GMT
```

web 服务器收到请求后发现有头 If-Modified-Since 则与被请求资源在客户端的最后修改时间 Last-Modified 进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），响应状态码为 HTTP 200；若最后修改时间 Last-Modified 较旧，说明资源无新修改，则 响应 HTTP 304 (这里只需要发送一个 head 头，包体内容为空，这样就节省了传输数据量)，告知浏览器继续使用所保存的 cache。

![if-modify.png](http://upload-images.jianshu.io/upload_images/4869616-b2ec0326e3e76b51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 其他

### Etag 和 Last-Modified

你可能会觉得使用 Last-Modified 已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要 Etag（实体标识）呢？HTTP1.1 中 Etag 的出现主要是为了解决几个 Last-Modified 比较难解决的问题：

1.  Last-Modified 标注的最后修改只能精确到秒级，如果某些文件在 1 秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
2.  如果某些文件会被定期生成，但内容并没有任何变化，但 Last-Modified 却改变了，导致文件没法使用缓存
3.  有些文档可能被修改了，但所做修改并不重要。（比如对注释或拼写的修改）
4.  有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

Etag 是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。Last-Modified 与 ETag 是可以一起使用的，<strong>服务器会优先验证 ETag，一致的情况下，才会继续比对 Last-Modified，最后才决定是否返回 304。</strong>

但是 Etag 也存在一些问题，比如：分布式系统尽量关闭掉 Etag(每台机器生成的 etag 都会不一样)。Etag 的服务器生成规则和强弱 Etag 的相关内容可以参考，《互动百科-Etag》,这里不再深入。

Last-Modified 和 ETags 请求的 http 报头一起使用，服务器首先产生 Last-Modified/Etag 标记，服务器可在稍后使用它来判断页面是否已经被修改，来决定文件是否继续缓存，

1.  客户端请求一个页面（A）。
2.  服务器返回页面 A，并在给 A 加上一个 Last-Modified/ETag。
3.  客户端展现该页面，并将页面连同 Last-Modified/ETag 一起缓存。
4.  客户再次请求页面 A，并将上次请求时服务器返回的 Last-Modified/ETag 一起传递给服务器。
5.  服务器检查该 Last-Modified 或 ETag，并判断出该页面自上次客户端请求之后还未被修改，直接返回响应 304 和一个空的响应体。

### Last-Modified/ETag 与 Cache-Control/Expires

如果检测到本地的缓存还是有效的时间范围内，浏览器直接使用本地副本，不会发送任何请求。两者一起使用时，Cache-Control/Expires 的优先级要高于 Last-Modified/ETag。即当本地副本根据 Cache-Control/Expires 发现还在有效期内时，则不会再次发送请求去服务器询问修改时间（Last-Modified）或实体标识（Etag）了。

一般情况下，使用 Cache-Control/Expires 会配合 Last-Modified/ETag 一起使用，因为即使服务器设置缓存时间, 当用户点击“刷新”按钮时，浏览器会忽略缓存继续向服务器发送请求，这时 Last-Modified/ETag 将能够很好利用 304，从而减少响应开销。

## 总结

![last1.png](http://upload-images.jianshu.io/upload_images/4869616-c7b6a0e81954a488.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![last2.png](http://upload-images.jianshu.io/upload_images/4869616-0b7a10398d925b86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 本文整理自[《HTTP 权威指南》之缓存详解](http://www.zyy1217.com/2017/05/14/HTTP%E7%BC%93%E5%AD%98%E8%AF%A6%E8%A7%A3/)

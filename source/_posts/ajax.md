---
title: ajax初步认识
date: 2017-09-13
tags: [ajax]
categories: http
---

## ajax 是什么

在学习 ajax 之前,如果想要发出 http 请求(form 表单或者输入地址回车),页面会整个刷新,极其影响用户体验.

我的理解就是 ajax 可以使得开发者在自定义的事件触发下进行 http 请求,因为是通过 js 完成的,可以在页面不进行刷新的情况下达到动态更新页面内容的效果.

AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。

AJAX 不是新的编程语言，而是一种使用现有标准的新方法。

AJAX 是在不重新加载整个页面的情况下,与服务器交换数据并更新部分网页的艺术。

百度的搜索推荐就是通过 ajax 向服务器发出请求,极少量的数据信息想要传递给服务器没必要刷新整个页面,ajax 就是实现在不进行页面刷新的情况下向服务器发出 http 请求.

## 如何实现 ajax

通过浏览器端的 js 帮我们预定义的一个异步对象来完成的 XMLHttpRequest.

XMLHttpRequest 是 AJAX 的基础。

发送 ajax 请求需要 5 步

1.  创建异步对象 XMLHttpRequest(兼容问题)

2.  设置请求的 url 等参数 open

3.  发送请求(请求报文,http 请求) send

    思考:为什么没有 get 方法
    因为网络请求是耗时的,只有通过某个事件的发生来获取信息,比如请求状态的改变
    如果存在 get 方法,什么时候调用?onreadystatechange?那和标准流程又有什么区别.

4.  注册事件(需要判断请求状态)

    `onreadystatechange` 事件。

    当请求被发送到服务器时，我们需要执行一些基于响应的任务。

    每当 `readyState` 改变时，就会触发 `onreadystatechange` 事件。

    `readyState` 属性存有 XMLHttpRequest 的状态信息。

    `onreadystatechange` 存储函数（或函数名），每当 `readyState` 属性改变时，就会调用该函数。

    `readyState` 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。

    0: 请求未初始化

    1: 服务器连接已建立

    2: 请求已接收

    3: 请求处理中

    4: 请求已完成，且响应已就绪

    status 200: "OK"

    404: 未找到页面

    在 onreadystatechange 事件中，我们规定当服务器**响应已做好被处理的准备时**所执行的任务。

    当 readyState 等于 4 且状态为 200 时，表示响应已就绪：

5.  在注册的事件中获取返回的内容并修改页面的显示

    数据是保存在异步对象的 responseText 属性中

    实际操作:

    - 通过某种条件发出 ajax 请求
    - 处理发过来的请求
    - 再回到浏览器异步对象 onreadystatechange 事件中 处理返回的内容

    上面我们是发送请求获得数据,倘若要发送数据到服务器端怎么办?

    ajax 通过 get 方法发送数据给服务器

    如果希望通过 GET 方法发送信息，请向 URL 添加信息：

    在 open 方法参数 url 中拼接要穿给服务器的数据 xxx.php?userName=jack(动态拼接)

    比如用户名注册重复检测:

    blur 时触发 ajax 事件,需要将输入的用户名传递给后台 php 页面,这时不是通过表单传递的,没有 submit,需要我们手动将数据传给后台,我们可以通过 js 获取用户输入的用户名,拼接在 url 后面发送请求.

    ```js
    xmlhttp.open('get', 'check.php?userName=' + un)

    xmlhttp.send()
    ```

    ajax 通过 post 方法发送数据给服务器

    然后在 send() 方法中规定您希望发送的数据：

    ```js
    xmlhttp.open('post', 'changeStar.php')

    xmlhttp.setRequestHeader(
      'Content-type',
      'application/x-www-form-urlencoded'
    )

    xmlhttp.send('starName=' + this.dataset['star'])
    ```

    注：通过 post 传递参数设置了请求头，确定发送信息至服务器时内容编码类型

    参见:[四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)

## 封装一个通用的 ajax 函数

```js
// data应为'a=a1&b=b1'这种字符串格式，在jq里如果data为对象会自动将对象转成这种字符串格式

function ajax_tool(url, method, data, success) {
  //异步对象
  var xmlhttp = new XMLHttpRequest()

  // 先注册异步回调事件，要在send方法之前
  xmlhttp.onreadystatechange = function() {
    if (xmlhttp.readyState === 4 && xmlhttp.status === 200) {
      success(xmlhttp.responseText)
    }
  }

  //url方法,get与post需要分别处理
  if (method === 'get') {
    if (data) {
      url += '?'
      url += data
    }
    xmlhttp.open(method, url)
    xmlhttp.send()
  } else {
    xmlhttp.open(method, url)
    xmlhttp.setRequestHeader(
      'Content-type',
      'application/x-www-form-urlencoded'
    )
    if (data) {
      xmlhttp.send(data)
    } else {
      xmlhttp.send()
    }
  }
}
```

tips: 还可以简化 ajax 函数,参数过多每次都叫记得参数顺序,可以直接传入一个对象,取得对象属性值即可.此处不做演示.

## 总结

1.  open(method, url, async) 方法

    method：发送请求所使用的方法（GET 或 POST）；
    与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用；

    然而，在以下情况中，请使用 POST 请求：

    - 无法使用缓存文件（更新服务器上的文件或数据库）
    - 向服务器发送大量数据（POST 没有数据量限制）
    - 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

    url：规定服务器端脚本的 URL(该文件可以是任何类型的文件，比如 .txt 和 .xml，或者服务器脚本文件，比如 .asp 和 .php （在传回响应之前，能够在服务器上执行任务）)；

    async：规定应当对请求进行异步（true）或同步（false）处理；true 是在等待服务器响应时执行其他脚本，当响应就绪后对响应进行处理；false 是等待服务器响应再执行( 会造成性能问题)。

2.  send() 方法可将请求送往服务器。

3.  onreadystatechange：存有处理服务器响应的函数，每当 readyState 改变时，onreadystatechange 函数就会被执行。

4.  readyState：存有服务器响应的状态信息。

    0: 请求未初始化（代理被创建，但尚未调用 open() 方法）

    1: 服务器连接已建立（open 方法已经被调用）

    2: 请求已接收（send 方法已经被调用，并且头部和状态已经可获得）

    3: 请求处理中（下载中，responseText 属性已经包含部分数据）

    4: 请求已完成，且响应已就绪（下载操作已完成）

5.  responseText：获得**字符串**形式的响应数据。

6.  setRequestHeader()：POST 传数据时，用来添加 HTTP 头，然后 send(data)，注意 data 格式；
    GET 发送信息时直接加参数到 url 上就可以，比如 url?a=a1&b=b1。

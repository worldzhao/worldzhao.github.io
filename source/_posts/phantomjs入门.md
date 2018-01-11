---
title: phantomjs入门
tags: [phantomjs]
categories: js
---

### PhantomJS 简介及使用
1. 用途
2. 网页自动控制
3. 网络监控
4. 获取网页截图
5. 无头测试

#### webpage模块

* open()

open方法用于打开具体的网页。

{% codeblock lang:javascript %}
    var page = require('webpage').create();

    page.open('http://slashdot.org', function (s) {
      console.log(s);
      phantom.exit();
    });
{% endcodeblock %} 

上面代码中，open()方法，用于打开具体的网页。它接受两个参数。第一个参数是网页的网址，这里打开的是著名新闻网站Slashdot，第二个参数是回调函数，网页打开后该函数将会运行，它的参数是一个表示状态的字符串，如果打开成功就是success，否则就是fail。

注意，只要接收到服务器返回的结果，PhantomJS就会报告网页打开成功，而不管服务器是否返回404或500错误。

open方法默认使用GET方法，与服务器通信，但是也可以使用其他方法。

{% codeblock lang:javascript %}
    var webPage = require('webpage');
    var page = webPage.create();
    var postBody = 'user=username&password=password';

    page.open('http://www.google.com/', 'POST', postBody, function(status) {
      console.log('Status: ' + status);
      // Do other things here...
    });
{% endcodeblock %} 

上面代码中，使用POST方法向服务器发送数据。open方法的第二个参数用来指定HTTP方法，第三个参数用来指定该方法所要使用的数据。

open方法还允许提供配置对象，对HTTP请求进行更详细的配置。

{% codeblock lang:javascript %}
    var webPage = require('webpage');
    var page = webPage.create();
    var settings = {
      operation: "POST",
      encoding: "utf8",
      headers: {
        "Content-Type": "application/json"
      },
      data: JSON.stringify({
        some: "data",
        another: ["custom", "data"]
      })
    };

    page.open('http://your.custom.api', settings, function(status) {
      console.log('Status: ' + status);
      // Do other things here...
    });
{% endcodeblock %} 

* evaluate()

evaluate方法用于打开网页以后，在页面中执行JavaScript代码。

{% codeblock lang:javascript %}
    var page = require('webpage').create();

    page.open(url, function(status) {
      var title = page.evaluate(function() {
        return document.title;
      });
      console.log('Page title is ' + title);
      phantom.exit();
    });
{% endcodeblock %} 

网页内部的console语句，以及evaluate方法内部的console语句，默认不会显示在命令行。这时可以采用onConsoleMessage回调函数，上面的例子可以改写如下。

{% codeblock lang:javascript %}
    var page = require('webpage').create();

    page.onConsoleMessage = function(msg) {
      console.log('Page title is ' + msg);
    };

    page.open(url, function(status) {
      page.evaluate(function() {
        console.log(document.title);
      });
      phantom.exit();
    });
{% endcodeblock %} 

上面代码中，evaluate方法内部有console语句，默认不会输出在命令行。这时，可以用onConsoleMessage方法监听这个事件，进行处理。

* includeJs()

includeJs方法用于页面加载外部脚本，加载结束后就调用指定的回调函数。

{% codeblock lang:javascript %}
    var page = require('webpage').create();
    page.open('http://www.sample.com', function() {
      page.includeJs("http://path/to/jquery.min.js", function() {
        page.evaluate(function() {
          $("button").click();
        });
        phantom.exit()
      });
    });
{% endcodeblock %} 

上面的例子在页面中注入jQuery脚本，然后点击所有的按钮。需要注意的是，由于是异步加载，所以phantom.exit()语句要放在page.includeJs()方法的回调函数之中，否则页面会过早退出。

* render()

render方法用于将网页保存成图片，参数就是指定的文件名。该方法根据后缀名，将网页保存成不同的格式，目前支持PNG、GIF、JPEG和PDF。

{% codeblock lang:javascript %}
    var webPage = require('webpage');
    var page = webPage.create();

    page.viewportSize = { width: 1920, height: 1080 };
    page.open("http://www.google.com", function start(status) {
      page.render('google_home.jpeg', {format: 'jpeg', quality: '100'});
      phantom.exit();
    });
{% endcodeblock %} 

该方法还可以接受一个配置对象，format字段用于指定图片格式，quality字段用于指定图片质量，最小为0，最大为100。

#### system模块

system模块可以加载操作系统变量，system.args就是参数数组。

{% codeblock lang:javascript %}
    var page = require('webpage').create(),
        system = require('system'),
        t, address;

    // 如果命令行没有给出网址
    if (system.args.length === 1) {
        console.log('Usage: page.js <some URL>');
        phantom.exit();
    }

    t = Date.now();
    address = system.args[1];
    page.open(address, function (status) {
        if (status !== 'success') {
            console.log('FAIL to load the address');
        } else {
            t = Date.now() - t;
            console.log('Loading time ' + t + ' ms');
        }
        phantom.exit();
    });
{% endcodeblock %} 

使用方法如下：

    $ phantomjs page.js http://www.google.com

#### 应用

* 加载网页 page.js

通过创建一个 webpage 对象，可以加载、分析和渲染网页。

{% codeblock lang:javascript %}
    // 创建 page 对象
    var page = require('webpage').create();
    // 打开特定的网页，抓取当前网页截图到本地
    page.open('http://example.com', function(status) {
      console.log("Status: " + status);
      if(status === "success") {
        page.render('example.png');
      }
      phantom.exit();
    });
{% endcodeblock %} 
*  测试加载速度 loadspeed.js

{% codeblock lang:javascript %}
    // address 不能少了 http 协议
    var page = require('webpage').create(),
      address = 'http://www.google.com',
      t, address;

    t = Date.now();

    page.open(address, function(status) {
      if (status !== 'success') {
        console.log('FAIL to load the address');
      } else {
        t = Date.now() - t;
        console.log('Loading ' + address);
        console.log('Loading time ' + t + ' msec');
      }
      phantom.exit();
    });
    运行 phantomjs loadspeed.js 即可看到如下内容

    Loading http://www.google.com

    Loading time 99 msec
{% endcodeblock %} 

*  获取网页内容

在打开一个网页后，我们往往有对其进行操作的需求，例如模拟点击登陆按钮、获取某个DOM元素等等，也就是需要在页面中执行javascript代码，这时候我们就需要使用到**evaluate()**方法。

由于因为evaluate()方法相当于一个沙盒，在其中是无法访问evaluate()之外的变量的，比如console.log就无法在evaluate中使用。

那如何将我想要获取的dom元素的id传进evaluate呢？
从PhantomJS 1.6开始，我们可以将外部变量以如下的方式传给evaluate内部，需要注意的是，能传入evaluate方法内部的参数只能是简单的基本类型，例如数值、字符串、json对象等能被JSON序列化的类型，而无法接受更复杂的对象，它的返回值也同样如此。

{% codeblock lang:javascript %}
    page.open('https://item.taobao.com/item.htm?id=520115087331', function(status) {
      var domId = "J_SellCounter"
      var sellCounter = page.evaluate(function(id) {
        return document.getElementById(id).innerText;
      }, domId);

      console.log(sellCounter);
      phantom.exit();

    });
{% endcodeblock %} 

由于open()方法打开的网页内部的console语句，和evaluate()方法中的console语句都不会执行，给我们开发调试带来了不便。这时可以采用onConsoleMessage回调函数，来打印出上面两种情况中的console语句中的信息：

{% codeblock lang:javascript %}
    var webPage = require('webpage');
    var page = webPage.create();

    page.onConsoleMessage = function(msg, lineNum, sourceId) {
      console.log('CONSOLE: ' + msg + ' (from line #' + lineNum + ' in "' + sourceId + '")');
    };
    其中msg是需要打印的信息，lineNum和sourceId是console.log在文件中的行号以及这个文件对应的标识id。

    var page = require('webpage').create();
    // 获取网页控制台的 console.log() 内容
    page.onConsoleMessage = function(msg) {
      console.log('Page console is ' + msg);
    };

    page.open('https://www.zhihu.com/', function(status) {
      var title = page.evaluate(function() {
        return document.title;
      });
      console.log('Page title is ' + title);
      phantom.exit();
    });
{% endcodeblock %} 

*  网络请求和响应

{% codeblock lang:javascript %}
    var page = require('webpage').create();
    // 打印请求
    page.onResourceRequested = function(request) {
      console.log('Request ' + JSON.stringify(request, undefined, 4));
    };
    // 打印响应
    page.onResourceReceived = function(response) {
      console.log('Receive ' + JSON.stringify(response, undefined, 4));
    };
    page.open('https://www.baidu.com/');
{% endcodeblock %} 

任务要求
编写一个task.js脚本，参考官网的includeJs方法，实现根据传入的参数（关键字），抓取百度第一页对应该关键字的搜索结果。

踩坑记录

* 控制台输出中文乱码 文件开头添加 phantom.outputEncoding = 'gbk'
* evaluate()方法中console报错
* dom操作没有在evaluate中进行
* 参数为中文是需要encodeURI一下

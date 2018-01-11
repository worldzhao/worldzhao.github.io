---
title: 记一次使用node+phantomjs+mongodb的任务
tags: [phantomjs,node,mongodb]
categories: js,db
---

### 网页抓取分析服务系列之三（服务封装）

#### 任务目的
1. 学习NodeJS HTTP模块
2. 学习NodeJS和本地进程的互动
3. 学习NodeJS和mongodb的交互

#### 任务描述
1. 安装nodejs和mongodb
2. 利用nodejs的HTTP模块封装一个node服务，监听8000端口，接受一个参数（关键字），http模块示例参考如下：

<pre>
  var http = require("http");
  http.createServer(function(request, response) {
      console.log('request received');
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write("Hello World");
      response.end();
  }).listen(8000);
  console.log('server started');
</pre>

3. 收到请求后，启动phantomjs进程执行taskjs，并将接受到的参数传递给phantomjs
4. phantomjs执行完后告诉node服务，并传回抓取的json结果
5. node服务将结果存到mongodb中（使用mogoose）

#### 任务注意事项
1. 参考nodejs和mongodb的相关文档快速学习和实践


#### 总结
相较于任务二，这次的任务综合性比较强，但是都是入门级的，譬如node,mongodb,phantomjs，都是api前两页的内容。前两次任务都是phantomjs简单入门，还好，翻翻文档就能过去，这次工具一多，杂了！

思路很简单：

1. 用node起一个http server，就访问localhost通过get方法在地址栏传关键字和设备名城区过去，因为http和url模块解析url太麻烦，所以使用了express，比较方便就可以取出参数。
2. 使用child_process模块的exec方法，拼接命令行启动phantomjs，让phantomjs去爬，爬完了将数据返回node
3. 将返回的数据存入数据库

很清晰！很简洁！很明了！对不对，尼玛我以为最大的坑在数据库(接触的太少了)，结果发现坑在node和phantomjs啊！！！！

##### 遇到的问题

1. 用phantomjs爬虫的时候最开始用的原生js操作dom,后来发现当改变设备的时候dom树也变了，就不能愉快的爬虫了。这时候发现可以用page.includeJs函数引入外部js，当然就要用jquery啦，我的jquery锋利锋利最锋利。然而includeJs的回调函数一直不执行，，原来这是一个异步操作，如果你在外面写了一个phantomjs.exit(),那你的jq还在加载的时候尚未进入回调操作dom，就直接运行到后面退出了phantomjs，所以phantomjs.exit()应当写在includejs回调函数内部。

+ includeJs()

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

2. 前面我们说了，当phantomjs爬虫爬完后，我们log一下整理好的json字符串，就可以被node的exec的回调函数获取(stdout)，然后就可以愉快的parse一下通过mongoose存入数据库啦，但是发现parse一直报错，说这个不是一个合法的json文件，然后就log了一下stdout，发现第一行是includeJs的log信息，就是jquery成功载入啦！然后我们的json字符串就没有那么的"json"了，试了很多方法，截取、正则替换、不知道为啥就是不能取出后半部分的json，发现stdout是个标准输出流，又去各种搜索，始终没有找到一个较好的处理stdout的方法，主要是我Node水平太差了，没有系统学习过，唉。后面不得已删除了includeJs，乖乖用原生js。

{% codeblock lang:javascript %}
    var exec = require('child_process').exec;
    var cmdStr = 'phantomjs task.js ';
    exec(cmdStr + queryObj.word + ' ' + queryObj.device, function(err, stdout, stderr){
         if(err) {
             console.error(`exec error: ${error}`);
         } else {
         // todo
         }
    })
{% endcodeblock %} 

##### 收获

1. 了解到node不仅仅是用库，那些基础概念需要深入了解。
2. 学习了mongodb的安装，启动以及mongoose的初步使用，知道了如何插入数据。初步理解了schema/model的概念。
3. 原来一直觉得用工具没有竞争力，现在突然觉得能把各种工具有机的结合在一起发挥生产力也是非常优秀的能力。
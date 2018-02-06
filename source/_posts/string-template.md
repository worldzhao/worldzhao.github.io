---
title: 简单实现一个字符串模版
date: 2017-09-13
tags: [原生js]
categories: js
---

> 先了解以下什么是字符串模版（模板引擎）http://www.jianshu.com/p/da57098f992a   
 目标：需要将字符串中所有使用花括号括起来的关键词，同义替换为对象字面量中对应的键值。

    字符串: '<a href={{href}}>{{text}}</a>'

    对象字面量:{href:'blog.hackerwen.tech' ,text:'我的博客'}

    结果: <a href="blog.hackerwen.tech">我的博客</a>

render.js

```
    function render(template,context) {
    // 准备正则 匹配至少一个 字母
    // 正则的 开始是 {{  结束是 }}
    // 中间的 小括号 可以对 正则 筛选出来的 字符串 再次筛选
      var reg = /{{(\w+)}}/;
    // 准备挖好坑的字符串
      var template = template;
    // 准备 用来填坑的 对象
      var context = context;
    // 首先 使用正则对象 验证一次 字符串 while 会看 result 是否有值
    // 这一次 找到的 有两个值
    /* 
        第一个  {{href}} 索引为0
        第二个 href  索引为1,小括号找到的
    */
      var result;
      while( result = reg.exec(template)){
          console.log(result);// 0:{{href}} 1:href
          // 获取 匹配的 key(href)
          var key = result[1];
          // 通过key 获取value
          var value = context[key];
          // 替换  替换的是 {{href}}
          template = template.replace(result[0],value);
      }
      // 执行完毕 
      return template;
    }
```
index.html
```
    //导入写好的函数
    <script type="text/javascript" src='render.js'></script>
    //定义模版
    <script type="text/template" id='template'>
        <a href="{{href}}">{{text}}</a>
    </script>
    <script type="text/javascript">
    // 通过标签获取模版
    var templateDom = document.querySelector("#template");
    var template = templateDom.innerHTML;
    console.log(template);//<a href="{{href}}">{{text}}</a>

    // 准备对象
    var context = {
        href:"http://blog.hackerwen.tech",
        text:"我的博客"
    }
    // 进行替换
    var resultStr = render(template,context);
    console.log(resultStr);
    // 添加到页面上
    document.body.innerHTML = resultStr;
    </script>
```
对正则表达式小括号使用方法不清楚的同学可以[戳这里](https://segmentfault.com/q/1010000004160141?_ea=511644)
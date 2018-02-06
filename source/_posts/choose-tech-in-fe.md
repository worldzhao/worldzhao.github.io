---
title: 高屋建瓴-前端技术选型
date: 2018-01-15
tags: [技术思考]
categories: 高屋建瓴
---
# 前端技术选型

> 该文章总结自：[慕课网: Webpack+React全栈工程架构项目实战精讲](https://coding.imooc.com/class/161.html)

高屋建瓴。

只写业务代码很难提升自己的前端能力，眼界要广要开阔，跳出自己当前的圈子。

先问自己一个问题。

## 我的项目需求是怎么样的

不同需求的项目适合不同的技术栈，传统的多页网站你不会选择React作为前端框架，单页应用选择jQuery来做也会变得非常困难，所以不同的项目适合不同的技术栈。

## 如何区分

### 两大分类

1. 多页应用
2. 单页应用

#### 多页应用

1. 特征：
    1. 内容都由服务端用模板生成
    2. 每次页面跳转都要经过服务端
    3. JS更多的只是做做动画，数据都是后端去处理

2. 常用类库(更多是dom api的封装类库，没有编程范式)
    1. jQuery
    2. mootools    
    3. YUI

3. 架构工具演进
    1. 无特定前端工具，跟后端配合
    2. grunt 处理任务的脚本处理器，每次从硬盘读取，效率较低
    3. gulp  采用stream流的形式，写法简洁，效率较高

4. 模块化工具
    1. 无。最初是没有模块化的，因为js用的少，基本是动效，一般就写在script标签中压缩一下了事，代码量很少。
    2. requirejs/seajs. web页面要求越来越高，重复代码越来越多，模块化需求出现, requirejs与seajs分别是对amd标准cmd标准实现，然而，现在用的更多的是commonjs标准，即node标准
    
5. 静态文件处理
    1. 使用gulp或grunt等工具手动编译到html中，自由度低，操作服务。甚至不处理，交给后端，让后端服务处理。    

#### 单页应用

1. 特征：
    1. 所有内容都在前端生成
    2. JS承担更多的业务逻辑，后端只提供API
    3. 页面路由跳转不需要经过后端

2. 常用类库(框架，开发者遵从其开发范式构建骨架)
    1. React
    2. Vue
    3. Angular
    4. ~~Backbone.js~~

3. 架构工具
    1. npm
    2. bower
    3. jspm

4. 模块化工具
    1. webpack
    2. rollup     `tree-Shaking`按需打包概念提出
    3. browserify

5. 静态文件

    可以直接在js代码中进行引用，并且交由模块化工具转化成线上可用的静态资源，并且可以定制转化过程适应不同的需求场景。

6. 存在的问题

    单页应用中所有的html内容都是通过js在浏览器进行生成的，我们在浏览器中输入一个url得到的是一个没有任何内容的html，要等待该html中引用的javascript代码加载完成之后才能渲染出来页面的内容，即我们开发时编写的jsx都是通过js代码生成html插入文档的，这就存在了几个问题：

    1. SEO不友好。单页应用加载过程中html是没有任何内容的，SEO无法抓取内容。
    2. 首屏时间较长，用户体验差。要等所有的javascript加载完成之后再去渲染页面，比直接加载html所耗费的时间长。

服务端渲染应运而生，依旧是React作为先驱，为我们带来了前后端同构的可能

要解决的问题有：

1. 数据同步
2. 路由跳转
3. SEO信息
4. 如何在开发时方便的进行服务端渲染的测试


## 其他考虑因素

1. 浏览器兼容性
2. toB还是toC
3. 移动端还是PC端

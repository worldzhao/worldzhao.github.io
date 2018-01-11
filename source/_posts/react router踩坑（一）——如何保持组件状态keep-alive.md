
---
title: react router(1)-实现keep-alive
date: 2017-10-05
tags: [React Router]
categories: js
---
## 前言

参考：
https://www.v2ex.com/amp/t/386516 
http://react-china.org/t/react-react-router/3757/2

最近在学习react，其实有过mvvm框架学习经验之后，再上手另一门框架不会太困难。
但是想要深入学习就会发现其中设计理念的不同，之前学的是vue，真的好方便，react初上手有些不习惯，习惯了jsx和es6写法之后，那可真是爽死了，比vue的模版简洁不少。

学习react router 4.0的时候将那些不适应放大到了极致，在RR4(react router 4.0)中，router是各种组件，对于习惯了之前vue的router.js配置文件的我来说，感觉画风一下子就变了。

这就是不同框架对于相同功能实现的不同理念吧，对于router组件化，这篇[聊聊 React Router v4 的设计思想](http://www.jianshu.com/p/e27cec8754ad)文章说的非常棒。

因为之前也用vue-router写过一些单页应用，上手RR4便马上寻找譬如路由嵌套、获取params、query等常用功能，看着文档也是能敲出来。

然而，如果真的那么轻松，我也不会在这浪费笔墨了。

写一个小demo的时候，想要实现类似于vue-router的keep-alive功能，突然发现，RR中没有，对的，就是没有。

keep-alive：包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

假如我有以下两种需求：

1. 有一个下拉加载更多列表页，当列表项达到1000项，点击列表项到详细页，当点击返回按钮时，保持列表页的一千条记录。
2. 把下拉加载更改为翻页，假设不是通过改变URL，只是单纯的ajax请求，击列表项到详细页，当点击返回按钮时，返回离开时的那一页。

最近再做一个知乎日报的demo，界面如下：

这是最新消息版块：
![image.png](http://upload-images.jianshu.io/upload_images/4869616-4603e90b36de5676.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

倘若最新是10月4号的新闻，当我翻页到10月2号，点进其中一篇文章，后退返回，发现我又回到了10月4号那一页。

这是主题日报版块：
![image.png](http://upload-images.jianshu.io/upload_images/4869616-47bef520c4897913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

倘若我选择“开始游戏”主题日报，当我点进其中一篇文章，后退返回，发现又回到了最初的“日常心理学”主题日报，很难受。

在vue里面可以很轻松解决的问题，在react里当然也可以解决，只不过要需要换一种方式。

### 方式一：
把加载页面的必要信息放URL里，后退就是浏览器的后退，回退时组件依然会重新渲染，componentDidMount的时候根据这些url的信息去请求API。数据保持离开时最新的状态。

这种方法在翻页情况下使用固然可以，但是像下拉加载那样，是多次请求的数据集合，就无法通过这种方式解决，而且后退时依旧进行了一次http请求。

### 方式二：
1. 存储当前已经加载的数据到本地（sessionStorage,localStorage）
2. 详情回退后,从本地取出数据，进行渲染（所以componentDid内部要判断本地是否已经存在数据，如果存在则return，不存在从服务器获取数据）

这种方式回退时不需要再次进行http请求，也能满足下拉的情况，我选择的就是这种方式。

如果还有更多的方法还望留言一同交流学习，谢谢回答过我这个问题的大佬！

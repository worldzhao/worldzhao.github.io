---
title: react router-模拟keep-alive
date: 2017-10-05
tags: [react-router]
categories: react
---

## 前言

参考：
https://www.v2ex.com/amp/t/386516
http://react-china.org/t/react-react-router/3757/2

最近在学习 react，其实有过 mvvm 框架学习经验之后，再上手另一门框架不会太困难。
但是想要深入学习就会发现其中设计理念的不同，之前学的是 vue，真的好方便，react 初上手有些不习惯，习惯了 jsx 和 es6 写法之后，那可真是爽死了，比 vue 的模版简洁不少。

学习 react router 4.0 的时候将那些不适应放大到了极致，在 RR4(react router 4.0)中，router 是各种组件，对于习惯了之前 vue 的 router.js 配置文件的我来说，感觉画风一下子就变了。

这就是不同框架对于相同功能实现的不同理念吧，对于 router 组件化，这篇[聊聊 React Router v4 的设计思想](http://www.jianshu.com/p/e27cec8754ad)文章说的非常棒。

因为之前也用 vue-router 写过一些单页应用，上手 RR4 便马上寻找譬如路由嵌套、获取 params、query 等常用功能，看着文档也是能敲出来。

然而，如果真的那么轻松，我也不会在这浪费笔墨了。

写一个小 demo 的时候，想要实现类似于 vue-router 的 keep-alive 功能，突然发现，RR 中没有，对的，就是没有。

keep-alive：包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

假如我有以下两种需求：

1.  有一个下拉加载更多列表页，当列表项达到 1000 项，点击列表项到详细页，当点击返回按钮时，保持列表页的一千条记录。
2.  把下拉加载更改为翻页，假设不是通过改变 URL，只是单纯的 ajax 请求，击列表项到详细页，当点击返回按钮时，返回离开时的那一页。

最近再做一个知乎日报的 demo，界面如下：

这是最新消息版块：
![image.png](http://upload-images.jianshu.io/upload_images/4869616-4603e90b36de5676.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

倘若最新是 10 月 4 号的新闻，当我翻页到 10 月 2 号，点进其中一篇文章，后退返回，发现我又回到了 10 月 4 号那一页。

这是主题日报版块：
![image.png](http://upload-images.jianshu.io/upload_images/4869616-47bef520c4897913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

倘若我选择“开始游戏”主题日报，当我点进其中一篇文章，后退返回，发现又回到了最初的“日常心理学”主题日报，很难受。

在 vue 里面可以很轻松解决的问题，在 react 里当然也可以解决，只不过要需要换一种方式。

### 方式一：

把加载页面的必要信息放 URL 里，后退就是浏览器的后退，回退时组件依然会重新渲染，componentDidMount 的时候根据这些 url 的信息去请求 API。数据保持离开时最新的状态。

这种方法在翻页情况下使用固然可以，但是像下拉加载那样，是多次请求的数据集合，就无法通过这种方式解决，而且后退时依旧进行了一次 http 请求。

### 方式二：

1.  存储当前已经加载的数据到本地（sessionStorage,localStorage）
2.  详情回退后,从本地取出数据，进行渲染（所以 componentDid 内部要判断本地是否已经存在数据，如果存在则 return，不存在从服务器获取数据）

这种方式回退时不需要再次进行 http 请求，也能满足下拉的情况，我选择的就是这种方式。

如果还有更多的方法还望留言一同交流学习，谢谢回答过我这个问题的大佬！

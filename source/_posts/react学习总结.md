---
title: 学习react，我学到了什么
date: 2018-02-02
tags: [React]
categories: js
---

# 学习react，我学到了什么

去年国庆节假期，我开始接触react。

之前学习过一段时间的vue。不过说实话，我vue学的不咋地，那是第一次从html + css + js的三板斧转移到极为先进和高贵的mvvm框架，众多概念让我有些眼花缭乱，我似乎掉进了海里。

vue的文档很完善，而那时候根基不稳，写出来的代码复用性差、模块划分乱、项目结构一团糟，以至于我把原来的demo都从github上删掉了，真是辣眼睛。

后面就到了秋招，针对面试学习了许多基础知识(这是重点，不得不说面试挑选的内容都是精华啊)。

那时候我到底学习了哪些对我学习现代前端开发有帮助的基础知识呢？

1. 多页应用与单页应用
    * 更为透彻的理解前后端分离
    * 后端路由与前端路由
2. 前端模块化
    * 闭包
    * AMD CMD commojs
    * ES6 Module
3. node
    * 前后端交互
    * express 与 koa 提供api或者进行api转发
    * 异步处理的演进（callback=>promise=>generator=>async）
    * MongoDB增删查改
4. webpack基础用法
    * webpack-dev-server
    * 热更新
5. ES 6常用新特性
6. event-loop

上面的知识(也许会有遗漏)为我学习react打下了坚实的基础，之前是为了学习vue而去学习vue，是因为存在vue-router|vuex所以我去使用vue-router|vuex，而现在我开始思考这些新事物为什么出现，又到底是为了解决了什么问题。

随着react的学习，一块一块黑暗的区域慢慢点亮。

vue让我接触到了诸多新事物，起码单页面应用、mvvm、前端路由、状态管理这四个概念算是有了一个初步认知。

[单页面应用与多页面应用](https://worldzhao.github.io/2018/01/14/%E9%AB%98%E5%B1%8B%E5%BB%BA%E7%93%B4-%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B/)

其实明确了这四个概念后，学习react很自然，我开始一边写demo一边学习react。

react本身并不难，无非就是使用jsx编写视图，重点在于掌握生命周期以及一些小坑，比如异步setState。

掌握了ES6语法会爽很多，诸如class、解构赋值、箭头函数、扩展运算符等等，react学习推荐看官方文档，结合余博伦同学以及阮一峰老师的入门教程（redux也可以看阮老师的）。

## 前端路由

因为并没有很复杂的数据状态需要去集中管理，所以我并没有学习并使用redux，但是使用了react-router，我学习的是最新的4.0版本，较之前版本改动颇大，当时十分不习惯react-route 4.0将路由组件一个一个分散在各处的做法。

后面阅读了一篇文章，详细地讲解了rr4.0的设计理念，才意识到，倘若没有接触过vue-router，那么可能我上手react-router 4.0速度会非常快，只要你会react，你就会react-router，一切都是那么理所当然，一切都是组件，和你编写的组件并没有什么不同。

[Build your own React Router v4
](https://worldzhao.github.io/2018/01/10/%E3%80%90%E8%BD%AC%E3%80%91%E6%89%93%E9%80%A0%E5%B1%9E%E4%BA%8E%E4%BD%A0%E8%87%AA%E5%B7%B1%E7%9A%84React%20Router%20v4/)

在学习这种前端路由库的时候，一同看了几篇关于前端路由的文章，对比了后端路由以及传统的服务端渲染，你可能不需要知道这些库的具体实现，但是知道一个大概的实现方式，对我们自身的眼界的开阔是很有帮助的。

前端路由：前端通过hash或者html 5的history api对url进行操作，浏览器内部消化url而不发送至后端，同时进行视图切换。

![react-router.png](http://upload-images.jianshu.io/upload_images/4869616-e627d9dcc4735111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[前端路由学习总结](https://worldzhao.github.io/2017/10/21/%E5%89%8D%E7%AB%AF%E8%B7%AF%E7%94%B1%E6%80%BB%E7%BB%93/)

对了，当初想找到类似vue-router的keep-alive的api，但是rrv4就是没有提供，还总结了一番。

[react router-模拟keep-alive](https://worldzhao.github.io/2017/10/04/react%20router-%E6%A8%A1%E6%8B%9Fkeep-alive/)


就这样在一边学习一遍操作的过程中完成了第一个demo，使用react做了一个知乎日报，没有学习redux，因为没有需要。

后面想起了之前写的一个音乐播放器，代码是真烂，而且数据状态比较复杂，刚好可以学习使用redux，就决定用react重构一遍，这次同时引入了eslint和editorconfig来规范自己的代码，配合vscode的插件格式化，天呐，代码终于不像屎一样。

[使用eslint和editorconfig规范代码](https://worldzhao.github.io/2018/01/16/%E4%BD%BF%E7%94%A8eslint%E5%92%8Ceditorconfig%E8%A7%84%E8%8C%83%E4%BB%A3%E7%A0%81/)

也是在这个项目实现过程中观摩了很多别人的目录结构(一个好项目的目录结构都能让人受益不少)，才知晓了如何去构建一个项目。

## 状态管理

那么现在来到了状态管理。

为什么需要状态管理呢？

在没有状态管理工具之前，我们只能通过props传递数据（不考虑context），可是某些信息是需要被许多组件公用的，并且这些组件不是简单的父子关系，用props十分繁琐，所以才需要状态管理工具，你可以理解为将那些需要公用的state变成了一个全局变量。

全局变量是魔鬼，于是redux设置了一些规则来规范开发者去变动state的行为，流程大致如图所示：

![redux.png](http://upload-images.jianshu.io/upload_images/4869616-af7401b4779dfce4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

“当你在考虑要不要上redux的时候，你其实并不需要redux”，这句话在我没有听过redux的时候就早有耳闻。

很多数据并不复杂的项目上redux只会让项目变得更为复杂，组件自身的state以及props如果已经足以完成需求，就没有必要使用redux。

当props已经无法满足你的欲望的时候，你应该会不由自主的想到redux，这个时候再去研究redux才是水到渠成，一切都很自然，不存在难不难的说法。

因为redux是一个很小的库，而且源码并不复杂，有很多源码分析文章，多看几篇，就熟练了，涉及到的设计模式是发布订阅模式。

至于中间件和异步action先不要着急，等你碰到需要配合redux发送异步请求的时候，就会自己去摸索。

[redux-simple-tutorial](https://github.com/kenberkeley/redux-simple-tutorial)

### 结合react-reudx

刚才没有谈到react-redux，react-redux只是一个辅助你在react中使用redux的库，背后还是redux那一套，但是我实在很想知道react-redux是怎么将redux套到react上的，看了一些文章，也算是初步了解。

在去深入理解react-redux前，你需要以下前置知识：

* react高阶组件 | ES6：修饰器 | 设计模式：装饰者模式 
* react context
* redux 原理 | 设计模式：发布订阅模式

[助你完全理解React高阶组件（Higher-Order Components）](http://react-china.org/t/react-higher-order-components/14949)

[ECMAScript 6入门：修饰器](http://es6.ruanyifeng.com/#docs/decorator)

[React文档：context](https://doc.react-china.org/docs/context.html)

然后你就可以大概去了解一下react-redux到底做了什么事情了，这里我是看胡子大哈的reactjs小书，虽然要10块钱，但是很值。

[React.js小书](http://huziketang.com/books/react/)

后面发现项目打包后首屏访问速度贼慢，又去了解了ssr，因为我还不是很会玩这个，就不扯淡了。

不过我清晰地了解到，前端这个海洋里，你下一步要喝什么水，你迟早都会知道。

这三个月的下来，项目也做了两个，不仅仅是初步掌握了react及其全家桶，并且对整个大局有了更为清晰的认识，思想跟上来了，就会好很多。

要多看别人的项目，多去慕课学习，不要一个人闷头学，走偏了很难拉回来。


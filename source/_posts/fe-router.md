---
title: 前端路由总结
date: 2017-10-22
tags: [原生js]
categories: js
---

>本文总结自：
* [Web开发中 前端路由 实现的几种方式和适用场景](http://blog.csdn.net/xllily_11/article/details/51820909)
* [HTML 5 History API的”前生今世”](http://blog.jobbole.com/78876/)
* [从零学习 React 技术栈系列教程](https://zhuanlan.zhihu.com/p/28769080)

最初接触前端路由是vue-router，当时仅仅觉得可以通过router-link改变页面，觉得很神奇呀，用多了就习惯了。

后面接触了node，通过express知晓了后端路由，这是我就开始疑问了：
为什么使用那些前端框架的时候，前端url改变了视图，但是却没有向后台发送请求？

首先我们需要一些前置知识。

## 路由/前端路由/后端路由？

路由：通过不同的url地址展示不同的内容或者页面。

前面提到过，我最初接触到路由是通过express框架，我们先看一段代码：
```js
app.post('/category/add', function (req, res, next) {
   // do something
})
```

如果你没有接触过node也没关系，因为真的不难理解。

定义了一个path('/categoty/add')，当有人通过(即域名+path)，例如：

http://www.example.com/category/add

发起post请求时，就会进入后台定义好的回调函数，进行逻辑处理，譬如取得post传递过来的实体数据，对数据库进行增删查改，然后返回一个渲染好的html页面或者是json数据等等。

后端路由（不考虑提供API服务返回数据），这一过程由服务器控制完成的，直接喷射一个html给前端，浏览器页面刷新。

那前端路由是什么呢？还是通过不同的url地址展示不同的内容或者页面，但是这一过程都是由前端完成的，我们的页面或视图（模块）是在前端编写好，通过url变化去切换而已。

为什么需要前端路由：

因为后台每次返回一个新页面都会进行全局刷新，而在单页面应用中，大部分页面结构不变，只改变部分内容的使用，我们可以通过前端路由改变页面内容，后台只需要通过ajax提供数据即可。

现在，我们来看看前端路由到底是如何实现的。

## 实现简易前端路由

首先我们需要了解HTML5为我们提供的history API，准确的说是这里面的两个方法:
history.pushState和history.replaceState

### history.pushState

>带有三个参数：一个状态对象，一个标题（现在被忽略了），以及一个可选的URL地址。下面将对这三个参数进行细致的检查：

实例代码（来自[Web开发中 前端路由 实现的几种方式和适用场景](http://blog.csdn.net/xllily_11/article/details/51820909)）

```js
//假设当前网页URL为：http://tonylee.pw
window.history.pushState(null, null, "http://tonylee.pw?name=tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw?name=tonylee

window.history.pushState(null, null, "http://tonylee.pw/name/tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw/name/tonylee

window.history.pushState(null, null, "?name=tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw?name=tonylee

window.history.pushState(null, null, "name=tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw/name=tonylee

window.history.pushState(null, null, "/name/tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw/name/tonylee

window.history.pushState(null, null, "name/tonylee");
//url变化：http://tonylee.pw -> http://tonylee.pw/name/tonylee

//错误的用法：
window.history.pushState(null, null, "http://www.tonylee.pw?name=tonylee");
//error: 由于跨域将产生错误
```

### history.replaceState

pushState()和replaceState()参数一样。
两个方法的主要区别就是：pushState()是在history栈中添加一个新的条目，replaceState()是替换当前的记录值。关于API的详细解释可以[戳这里](http://blog.jobbole.com/78876/)

我们需要知道的是，就是两个方法可以改变浏览器的url，但是不会重新加载页面，没错！！！这些 URL 不会直接传给服务器，而是会被浏览器消化处理掉.

这个就是我们需要的。

这样我们就可以通过这两个方法改变url了。

改变之后呢？切换视图呀！

那我们就需要在这两个方法被调用的时候，触发一个方法去切换视图，好在HTML5也给我们提供了一个事件。

### window.onpopstate

浏览器本身会自带一个popstate事件，但是只有在我们点击返回或前进按钮时才会正常触发。

很显然，在我们单页应用中是需要你去大部分情况下都是需要去点击某个Link调用pushState，而这样是无法触发popstate事件的，需要重写一下pushState，并且给他也定义一个事件，这里就叫他onpushstate吧，比如下面这样。

```js
(function(history){
    var pushState = history.pushState;    
    history.pushState = function(state) {
        if (typeof history.onpushstate == "function") {            
            history.onpushstate({state: state});        
        }
        return pushState.apply(history, arguments);   
    }
})(window.history)

window.onpopstate = history.onpushstate = function(event) {
    // change view
}
```

如此一来，进行pushState操作会触发onpushstate事件，我们可以在onpushstate事件的回调中进行视图切换了，而前进后退我们可以通过popstate来操作。

别忘了给a标签做一些必要操作，阻止默认跳转，而是通过pushState改变url，然后由于pushState被调用，又会触发onpushstate事件，其内的逻辑代码就会被执行，比如，切换视图。

```js
var elements = document.getElementsByTagName('a');
for(var i = 0, len = elements.length; i < len; i++) {    
    elements[i].onclick = function (event) {        
        event.preventDefault();
        var route = event.target.getAttribute('href');        
        history.pushState({page: route}, route, route)
    }
}
```

前端路由的基本实现就是以上了，代码来自[余博伦-知乎首页](https://www.zhihu.com/people/yubolun/activities)，现在再回头看vue-router/react-router是否清晰了一点呢，当然其内部实现远远不是这么简单。

## 小知识-pjax

pjax是一种基于ajax+history.pushState的新技术，该技术可以无刷新改变页面的内容，并且可以改变页面的URL。pjax是ajax+pushState的封装，同时支持本地存储、动画等多种功能。目前支持jquery、qwrap、kissy等多种版本。

众所周知，Ajax可以实现页面的无刷新操作——优点；但是，也会造成另外的问题，无法前进与后退！曾几何时，Gmail似乎借助iframe搞定，如今，HTML5让事情变得如同过家家般简单。

当执行Ajax操作的时候，往浏览器history中塞入一个地址（使用pushState）（这是无刷新的）；于是，返回的时候，通过URL或其他传参，我们就可以还原到Ajax之前的模样。

[ajax与HTML5 history pushState/replaceState实例-张鑫旭](http://www.zhangxinxu.com/wordpress/2013/06/html5-history-api-pushstate-replacestate-ajax/)
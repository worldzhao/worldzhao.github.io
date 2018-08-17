---
title: ZhihuDaily-React项目总结
date: 2017-10-09
tags: [项目实战]
categories: react
---

API：[点这里，感谢](https://github.com/izzyleung/ZhihuDailyPurify/wiki/%E7%9F%A5%E4%B9%8E%E6%97%A5%E6%8A%A5-API-%E5%88%86%E6%9E%90)

项目地址：[Github](https://github.com/hackerwen/ZhihuDaily-React)

![pc主页.png](http://upload-images.jianshu.io/upload_images/4869616-fa384335b62c2ea2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![移动端主页.png](http://upload-images.jianshu.io/upload_images/4869616-b3d1a55cc163bb0e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![内容详情.png](http://upload-images.jianshu.io/upload_images/4869616-f28324cf41e6b038?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![评论.png](http://upload-images.jianshu.io/upload_images/4869616-ff91fbcaadcbab49?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 依赖

- 框架：react
- 路由：react-router 4.0
- HTTP 请求：whatwg-fetch(主要是保证 fetch 兼容性)
- 设备判断：react-responsive
- UI：antd(主要是 PC 端)
- CSS 预处理：stylus
- 移动端布局：flexible.js + rem

## 遇到的问题以及解决方法

### API 问题

#### 1. API 不支持跨域

那位大大的 API 是不支持跨域的，于是自己使用 koa2+request-promise+koa-cors 进行了后台 API 转发 ，代码如下，在源码的 server/server.js 中

```js
const Koa = require('koa')
const cors = require('koa-cors')
const rp = require('request-promise')
const app = new Koa()
const basrUrl = 'http://news-at.zhihu.com'

app.use(cors())

const main = async (ctx, next) => {
  const pathname = ctx.request.path
  ctx.response.type = 'json'
  ctx.response.body = JSON.parse(await rp(basrUrl + pathname))
}

app.use(main)

app.listen(9999)
console.log('app started at port 9999...')
```

启动了服务后，对自己的本地服务器进行请求即可，只需要将原 API 的“http://news-at.zhihu.com”改为“http://localhost:9999”。

#### 2.知乎图片防盗链问题

“知乎的图片可能会通过请求头的 referer 参数判断，如果不是指定的域名会返回 403，如果精通后台的同学，可以去访问这些图片来缓存这些图片，我是搜索到了一个相对简单一些的办法，点击链接，主要用到的是 Images.weserv.nl 这个网站，可以缓存图片，而且可以修改图片的尺寸大小。

具体是我自定义了一个过滤器，在需要用到地方，把知乎的 url 替换成这个图片代理网站的 url，这样图片就可以显示了。”

```js
const replaceUrl = srcUrl => {
  return srcUrl.replace(/http\w{0,1}:\/\/p/g, 'https://images.weserv.nl/?url=p')
}
```

然后对返回的 data 直接进行如下操作:

```js
data = JSON.parse(replaceUrl(JSON.stringify(data)))
```

data 中的图片就可以正常引用了。

> 参考
>
> - [使用 vue 完成知乎日报 web 版](http://www.yatessss.com/2016/07/08/%E4%BD%BF%E7%94%A8vue%E5%AE%8C%E6%88%90%E7%9F%A5%E4%B9%8E%E6%97%A5%E6%8A%A5web%E7%89%88.html)
>   这个博主做的 vue 版十分完善，使用 vue 的同学可以看看。

### React/React Router4.0 相关问题

#### 1.React Router 为何在 url 相同参数不同的情况下跳转但是并不刷新页面？

案情描述：当我从“日常心理学”主题日报跳转至“用户推荐日报”时，组件内部数据没有刷新，仍然为“日常心理学”的数据，而非“用户推荐日报”数据。

路由一：/topic/abc

路由二：/topic/efg

即路由一跳转至路由二组件没有重新去获取数据，其实这个问题在 vue-router 里面也有，vue 中是通过 watch 监测路由参数变化判断组件是否需要刷新。

解决方案：我们一般是在 componentDidMount 中获取数据，当我从路由一跳转至路由二，仍然为同一组件，所以并没有重新 Mount。

也就是说，同级路由相同组件跳转的情况下，componentDidMount 方法仅仅在第一次 Mount 的时候触发了，路由跳转并没有触发该方法。

但是路由跳转之后 props.params 变了, 我们可以在 componentWillReceiveProps 中进行第二次的数据拉取，于是会触发更新过程。

> 参考：
>
> - [React Router 为何在 url 相同参数不同的情况下跳转但是并不刷新页面？](https://segmentfault.com/q/1010000009309279)

#### 2. 如何保持路由组件 keep-alive，跳转返回后不重新渲染

案情描述：假如我有以下两种需求：

1.  有一个下拉加载更多列表页，当列表项达到 1000 项，点击列表项到详细页，当点击返回按钮时，保持列表页的一千条记录。
2.  把下拉加载更改为翻页，假设不是通过改变 URL，只是单纯的 ajax 请求，击列表项到详细页，当点击返回按钮时，返回离开时的那一页。

其实我之前也写过篇文章，[戳这里](http://www.jianshu.com/p/869e5c2e45cb)

某一天我很无聊，正在刷知乎日报，我要完成一个目标：从 2017-10-07 号一直看到 2017-01-01。

我一直看到了 10-05 的信息，此时列表已经较长了，点进去，看完了，返回，组件重新渲染，滚动条还原，数据还原，我又下拉到三天前，又看了一篇文章，返回，重复上述操作。

随着列表越来越长，我每一次返回的代价都是巨大的，我都要重新拉回到之前看到的地方，且数据还要重新加载，因为每次返回只能返回 latest 信息，无法返回你通过 ajax 下拉加载 beforeNews 的信息。

很显然，这个问题不解决，用户体验绝对负分。

解决方案：

方式一：

把加载页面的必要信息放 URL 里，后退就是浏览器的后退，回退时组件依然会重新渲染，componentDidMount 的时候根据这些 url 的信息去请求 API。数据保持离开时最新的状态。

这种方法在翻页情况下使用固然可以，但是像下拉加载那样，是多次请求的数据集合，就无法通过这种方式解决，而且后退时依旧进行了一次 http 请求。

方式二：

1.  在 componentWillUnMount 方法中存储当前已经加载的数据到本地(sessionStorage,localStorage)
2.  详情回退后,从本地取出数据，进行渲染(所以 componentDidMount 内部要判断本地是否已经存在数据，如果存在则 return，不存在从服务器获取数据)
3.  这种方式回退时不需要再次进行 http 请求，也能满足下拉的情况，我选择的就是这种方式。

> 参考：
>
> - [react-router-dom 怎么让第二个页面返回到第一个页面使得第一个页面不重新加载](https://www.v2ex.com/amp/t/386516)
> - [React 和 react-router ,实现回退的时候，如何使页面回退到以前的状态](http://react-china.org/t/react-react-router/3757/2)

#### 3.后退时记录滚动条位置

案情描述：同上

恢复数据还不够，只是省下了我加载的时间，而我还是要拉到最底部，所以滚动条的位置也需要恢复。

思路同上，将当前的 scrollTop 记录在一个全局变量中(我是作为 LatestNews 组件的属性，为了让需要滚动的组件获取到)，在 componentWillUnMount 方法中存储当前的滚动条位置(sessionStorage,localStorage)。

当后退返回时取出位置赋值给 LatestNews.scrollPoint，确认数据更新后(一定要确认数据更新后，setState 方法是异步方法)，在 setState 的回调中根据 Latest.scrollPoint 滚动需要滚动的组件。

> 参考
>
> - [react 判断滚动到底部以及保持原来的滚动位置](http://blog.csdn.net/tujiaw/article/details/77511460)

#### 4. 当 state 的属性为数组，如何 setState 比较好

这个主要是 push,unshift 方法的返回值造成 state 没有赋值成功，最好先用 slice 把数组 copy 以下，用 copy 的数组去 push(data)，然后再去 setState.

```js
const _storiesQue = this.state.storiesQue.slice()
_storiesQue.push(data)
this.setState({
  storiesQue: _storiesQue
})
```

> 这里有一篇文章写的很好
>
> - [深入理解 React 组件状态（State）](http://www.jianshu.com/p/c6257cbef1b1)

### 移动端适配问题

#### 1.react-responsive

```bash
npm install  react-responsive --save
```

然后在 index.js 中

```bash
import MediaQuery from 'react-responsive';

ReactDOM.render(
  <Router>
        <div>
            <MediaQuery query='(min-device-width:1224px)'>
                <PCIndex/>
            </MediaQuery>
            <MediaQuery query='(max-device-width:1224px)'>
                <MBIndex/>
            </MediaQuery>
        </div>
    </Router>,
  document.getElementById('root')
);
```

在设置了 query 参数后，就可以根据设备的宽度来决定渲染 PC 组件还是移动端组件，独立开发即可，当然，要考虑组件的复用。

#### 2.flexible.js+rem 布局

关于 flexible.js+rem 布局有疑问的可以看下面的文章

> 参考：
>
> - [移动端高清、多屏适配方案](http://div.io/topic/1092)
> - [使用 Flexible 实现手淘 H5 页面的终端适配](https://github.com/amfe/article/issues/17)

使用 lib-flexible.js

首先:

```bash
npm install lib-flexible --save
```

然后在 index.js 中，即可使用 rem 进行适配了。

```bash
import 'lib-flexible';
```

将 px 转为 rem 的几种方法

1.  可以通过开发工具的插件，例如 sublime 上面就有插件
2.  webpack 的 px2rem-loader，对 stylus 文件无效
3.  使用定义好的预处理器(stylus,sass,less)的方法

以下是 stylus 的示例

```stylus
//定义一个变量和一个mixin
$baseFontSize = 16; //默认基准font-size
px2rem(name, px){
{name}: px / $baseFontSize * 1rem;
}

// 使用示例：
.container {
  px2rem('height', 240);
}

//  stylus翻译结果：
.container {
  height: 3.2rem;
}
```

> 参考：
>
> - [三种预处理器 px2rem](http://www.cnblogs.com/shahramLu/p/6408551.html)

#### 2. Drawer 组件遮罩层滑动组织底部窗体滚动

因为 PC 端用的是 ant-design，对移动端不是很友好，所以移动端基本是自己手写(除了引入了 antd 的轮播图(Carousel)和折叠面板(Collapse)，不过真的丑)。

Drawer 组件，如下图所示:

![Drawer.gif](http://upload-images.jianshu.io/upload_images/4869616-32fec349e60cd197?imageMogr2/auto-orient/strip)

设计思路是：Drawer 组件先通过绝对定位移出视窗外，当点击 menu 按钮时，添加 class，改变 left 以及 top 的值,当然得有 transition 营造动画效果(还可以通过 transform:translateZ(0)硬件加速)，同时渲染遮罩层 mask，宽高均为 100%.

z-index 层级：Drawer>mask>index

案件描述：当我不小心滑动到遮罩层 mask 的时候，底部 list 也在滑动，这显然不是我想要的，这里主要是看了张鑫旭大佬的文章。我就不细说了。

> 参考
>
> - [web 移动端浮层滚动阻止 window 窗体滚动 JS/CSS 处理](http://www.zhangxinxu.com/wordpress/2016/12/web-mobile-scroll-prevent-window-js-css/)

### 关于组件复用

移动端以及 PC 端的 news_detail/comments 均是基于公共的模块 news_content/comments_content/comment/avatar 进行二次包装。

以内容详情页 news_detail 举例。

PC 端内容详情页 news_detail 这一模块中包含评论 comments 模块，而移动端包含一个 news_header 头部模块。

所以采取对共有模块 news_content 进行引用，同时 PC 端引用 comments 模块进行组合，移动端引用 news_header 模块进行组合，在各自的 news_detail 里进行数据的获取，共有模块只负责展示，属于 stateless 组件。

这样一来，即使完全删除 PC 文件或者移动端文件，剩下的一方依然可以正常使用，二者之间不存在相互关联引用。

### 关于打包(未解决)

记得要将开发时对 webpack.config.dev.js 做的操作也要对 webpack.config.prod.js 操作一遍。

比如我开发时引入了 stylus，自己也配置了 webpack.config.dev.js 的环境[stylus, stylus-loader ]，在开发环境下没有问题。

但是打包之后 stylus 文件全部失效，一看好好的躺在 static/media 文件夹呢，说明生产环境的 webpack 没有配置好。再对 webpack.config.prod.js 里面的 loaders 也添加一遍就好了，其它同理。
然后：

```bash
npm run build
```

如何在 github-page 上展示呢？

[戳这里](https://segmentfault.com/a/1190000008425992)，虽然说的是 vue 项目，但是 build 后操作一模一样，除了将命令中 dist=>build

上传后会发现只有在 github 根路径才能正常访问，而不是仓库下面的路径，很坑啊。。。不是很懂这个原因啊，恳请高手替我解答一下，资源路径没问题，就是路由匹配不正确。

理想情况下应该是这个路径可以直接访问

```js
https://nickname.github.io/project/[index.html]
```

但是这个 gh-pages 的的路径进去是一个 header（PC 端），路由'/'的组件没有渲染出来，点了以下 header 的首页'/topics'的链接，一切恢复正常，但是 url 却变成了下面这个：

```js
https://nickname.github.io/topics
```

项目路径被吃了，这个路径直接访问又是 404，难道 react-router 写 path 的时候只能相对于域名根路径？

我看别人都是打包后资源加载问题，我发现直接打包资源路径没有问题，难道我开发时要把 path 由'/a'改为'/project/a'???

如果有成功在 gh-pages 上展示项目的同学，请务必在评论中留下您的解决方案，真的非常感谢！

> 参考
>
> - [stackoverflow](https://segmentfault.com/q/1010000009672497)
> - [如何在 gh-pages 上展示 vue 项目](https://segmentfault.com/a/1190000008425992)

后续操作：完善细节，自己写折叠面板。

## 后话

如果我的文章给您了一点帮助，希望可以去 github 上给我一颗 star。

能够在国庆短短几天从不知道 react 是啥到做个小东西出来，一直都是因为有这么多热爱分享的人，站在巨人的肩膀上！

但是仍然有很多不足，请指正。

项目地址：[狂戳这里](https://github.com/hackerwen/ZhihuDaily-React)
⭐⭐⭐

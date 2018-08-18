---
title: 前端性能优化
date: 2017-09-24
tags: [性能优化]
categories: 工程化
---

![性能优化思考维度.jpg](http://upload-images.jianshu.io/upload_images/4869616-36154be394a65dfd.jpg?imageMogr2/auto-orient/strip)

> 性能优化感觉大家都能说出几条来，粗略点无非是减少 http 请求、减轻请求数据大小等等，详细点就是 css/js 合并压缩、雪碧图等等，但实在是散乱无章，如何有条理地回答面试官的问题就很重要了，其实可以从不同的维度，不同方面去进行回答。
> 总结自：
> [鸟瞰前端 , 再论性能优化](https://juejin.im/post/59c2109cf265da066875eff5)

## 预备知识

### “从用户输入 URl 到页面展示给用户浏览器客户端的过程中发生了什么？”

![http.jpg](http://upload-images.jianshu.io/upload_images/4869616-d4ecfc72af5e399c.jpg?imageMogr2/auto-orient/strip)
[前端经典面试题: 从输入 URL 到页面加载发生了什么？](https://segmentfault.com/a/1190000006879700)

### 浏览器渲染原理

![页面展示过程.png](http://upload-images.jianshu.io/upload_images/4869616-597251687b12933e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  HTML 被解析成 DOM Tree，CSS 被解析成 CSS Rule Tree
2.  把 DOM Tree 和 CSS Rule Tree 经过整合生成 Render Tree（布局阶段）
3.  元素按照算出来的规则，把元素放到它该出现的位置，通过显卡画到屏幕上

CSS 不会阻塞 DOM Tree 生成，会阻塞渲染，但 JS 会阻塞 DOM 加载。

### 重绘和回流

[如何写出高性能 DOM？](http://34585f3f.wiz03.com/share/s/0Qm5Y_0RRQtc2F-3Zy2piy1K0E4QKp0IAQvZ2PEFvB08u3fM)

## 浏览器宿主环境

### 1. 突破单线程解析渲染阻塞限制

浏览器是一个单线程解析模式去解析渲染从服务器端拿到的 html 文本，css 加载（渲染）的过程中会对后续的脚本资源加载造成阻塞，脚本的加载也会阻塞后续 DOM 结构的解析造成页面的留白时间增长，雅虎的 35 条军规中有一条就是**样式文件放在头部，脚本文件放在 DOM 节点最末尾**，减少阻塞。这里还有几个针对脚本文件的优化：

- 针对不需要 DOM 操作（主要考虑是需要操作 DOM 的脚本往往需要获取一些样式信息）的 Js 脚本可以采用动态创建 script 的方式载入，动态载入的脚本不阻塞后续资源的加载。
- 脚本文件加载可以加上 defer 或者 async 属性标识防止阻塞
  defer 是在 HTML 解析完之后才会执行，如果是多个，按照加载的顺序依次执行
  async 是在加载完成后立即执行，如果是多个，执行顺序和加载顺序无关

### 2. 避开 Cookie 性能 bug——静态资源 CDN

Cookie 是前端作为前后台登录态校验最通常用的缓存方案，但鉴于浏览器在每次都会往同域的任何资源的 http 请求中自动带上 cookie 信息的情况，这里有必要进行优化一下，因为像 css、js、image 这些资源请求是不需要 cookie 信息的，会无端造成请求带宽的浪费（想象一下我们的 cookie 大小假设为 10K，100 个请求就是近 1M 的大小，高并发下以我们现行网络带宽也是蛮大的一笔负担了）。

Cookie free 性能优化方案的处理方式是 CDN 异域静态资源服务器部署我们的前端 css、js、image 资源。

### 3. 代码优化——事件委托等

其实这里有很多点：

- 要插入 DOM 片段时最好使用 fragment 一次性插入；
- 动态生成 100li 如何绑定事件，可以通过事件冒泡利用事件委托给父元素绑定事件即可；
- 对于查找过的节点可以缓存下来，避免再次查找
- ...

## DNS 层

### DNS 预解析

[前端优化:DNS 预解析提升页面速度](http://skyhome.cn/div_css/301.html)
这里也是我第一次遇到 link 标签的另外用法。

## HTTP 层

### 1. 减少 HTTP 请求数量

1.  合并 CSS、JS 文件
2.  CSS sprites 雪碧图
3.  font-icon 字体图标
4.  图片 base64 编码传输
5.  [图片懒加载](http://www.jianshu.com/p/4876a4fe7731)
6.  [HTTP 缓存机制](http://blog.hackerwen.tech/2017/09/14/HTTP%E4%B9%8B%E7%BC%93%E5%AD%98/)

### 2. 减轻 HTTP 请求数据大小

1.  css、script、图片压缩：这些可以 gulp 或者 webpack 自动化脚本里面定义脚本任务来完成。

2.  服务器开启 gzip 压缩：一般现在服务器都有开启 Gzip 压缩，压缩率通常都是 30%以上，效果还是不错的。

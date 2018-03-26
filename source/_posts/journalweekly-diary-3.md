---
title: 【周记】2018.03.19-2018.03.25
date: 2018-03-25
tags: [周记]
categories: 琐事
---

关键字：
1. 需求提测
2. 阅读zent-alert/swiper源码
3. react-tiny-swiper
4. npm link
5. npm publish
6. 使用gh-pages展示react项目
7. mix content错误
8. webpack

## 工作
1. 订阅管理提测-修复。
2. 采购单位新增驻外地信息字段 开发-提测-修复-发布。这个feature让我见到了代码能够有多乱，写业务代码倘若为了复用行丧失了维护性，那可真是糟糕。不过写得好的代码也很难应对无止尽的需求，尽量拆的**越小**越好，抽离出公共的，不要混杂在一起，单一职责。
3. 这周六就可以回学校了，哈哈哈哈哈

## 学习
1. 封装了 `react-tiny-swiper` 组件并发布到了npm。学会了如何编写组件，搭建组件开发环境，开发打包测试一条龙服务。
    - [发布react组件到npm总结](https://worldzhao.github.io/2018/03/23/publish-your-first-react-component-to-npm/)
    - [react-tiny-swiper](https://github.com/worldzhao/react-tiny-swiper)
2. 重新老老实实的学习webpack，理一遍概念，在看慕课视频的同时看这篇文章-[从零搭建React全家桶框架教程](https://github.com/brickspert/blog/issues/1#react-router)。
3. 利用`gh-pages`将music-react在线展示。不需要同原来一样去修改 `publicPath` ，新建 `gh-pages` 分支了，大致操作如下(除了第一步，你build之后react都会告诉你如何操作)：
    - `BrowseRouter` 添加 `basename`属性
      ```js
      <BrowserRouter basename="/music-react/">
      ```
    - `package.json` 文件添加 `homepage` 字段
      ```js
      "homepage": "http://worldzhao.github.io/music-react",
      ```
    - 安装 `gh-pages` 包
      ```
      npm install gh-pages --save-dev
      ```
    - `package.json` 添加 `predeploy` / `deploy` script命令
      ```js
      "scripts": {
        "start": "node scripts/start.js",
        "build": "node scripts/build.js",
        "test": "node scripts/test.js --env=jsdom",
        "predeploy": "npm run build", // 新增
        "deploy": "gh-pages -d build" // 新增
      },
      ```
    - 执行命令
      ```
      npm run build # 或npm run predeploy
      npm run deploy
      ```
4. 在进行3的过程中，发现 `gh-pages` 是https网页，里面禁止http请求（css,js,img会报warning,其他请求ajax等直接block报error）。[music-react](https://worldzhao.github.io/music-react/)，如下图：
![error.png](https://upload-images.jianshu.io/upload_images/4869616-988a4e2192d33543.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![warning.png](https://upload-images.jianshu.io/upload_images/4869616-9ead2522991a91ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原来展示是可以的，因为使用的是自定义域名`http://blog.hackerwen.tech`，没有使用`https://worldzhao.github.io`这个域名，要么重新自定义域名，要么升级服务到https。
5. 在编写 `react-tiny-swiper` 过程中看了一下zeng-alert/swiper源码，学习到了一些东西。
    - [让繁琐的if else逼格高一点点](https://mp.weixin.qq.com/s/cInFsWjCRGtKnZ17IfFXUw)，这是我很久之前看的一篇文章，在自己也在践行，zent中处理样式的时候也是这种方式
    - 代码尽可能的小，这个是我最大的收获了。
    改动前：
    ```js
    window.onblur = () => {
      clearInterval(this.autoTimer)
    }
    ```
    改动后：
    ```js
    stopAutoPlay = () => {
      clearInterval(this.autoTimer)
    }

    window.onblur = () => {
      this.stopAutoPlay()
    }
    ```
    通过方法名去自己解释自己，不用再去细看代码是如何实现的了，阅读性大大增加，虽然自己一直知道这一点，但是对比两份代码，羞愧的低下了头。

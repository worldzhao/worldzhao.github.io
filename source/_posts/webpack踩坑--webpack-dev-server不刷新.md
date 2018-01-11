---
title: webpack踩坑（一）——webpack-dev-server不更新
tags: [webpack]
categories: webpack
---
## 前言
第一次接触webpack是使用vue-cli的时候，方便极了，一行命令chua！chua！chua！
好嘞，上手吧。

而对我这种新人来讲，需要手动搭建webpack开发环境的时候就是一脸懵逼了。

当你觉得一个工具很自然的时候，说明你还没有要到使用这个工具的地步。

只有经过了各种折磨之后，有人突然给你看一个大宝贝儿：“喏，这个很方便的”。你惊为天人的时候，这个工具才适合你。否则只是一个搬砖工，知其然不知其所以然。当然，搬砖搬得6那也是项绝活儿。

基本概念就不提了，什么entry入口文件、output输出路径、loader干啥的、plugin又是啥，看英文单词的中文意思即可。

## 正文
今天在自己搭建react开发环境的时候，才知晓以前vue开发时自动更新是如何做到的，当然踩了不少坑，以下代码均是在webpack 3.3.0环境下操作。

### webpack-dev-server

#### 安装
webpack-dev-server就是起一个小型服务器，底层是express。
```bash
npm install webpack-dev-server --save-dev
```
#### 配置
* 使用babel-loader以及各种相关的preset解析我们的react语法jsx以及es语法;
* 使用css-loader以及style-loader让我们能够在jsx中引入css文件;
* 使用file-loader以及url-loader对图片进行处理（还有坑爹的路径问题，下一篇文章写）。
代码如下：
```js
var webpack = require('webpack');
var path = require('path');

module.exports = {
    entry: {
        app: path.join(__dirname, 'src', 'index.js')
    },
    module: {
        loaders: [
            { // js的loader，可以理解为gulp里面对js文件进行操作的插件，此处是依赖babel-loader将我们所编写的es6以及jsx转变为浏览器可以识别的代码
                test: /\.js?$/,
                exclude: /(node_modules)/,
                loader: 'babel-loader',
                query: {
                    presets: ['react', 'es2015', 'stage-0'],
                    plugins: ['react-html-attrs'],// 组件的插件配置，jsx可以不用写className，写class，其实意义不大
                }
            },
            {
                test: /\.css?$/,
                loader: 'style-loader!css-loader'
            },
            {
                test: /\.(png|jpg)$/,
                loader: 'url-loader?limit=8192&name=images/[hash:8].[name].[ext]'
            }
        ]
    },
    plugins: [ // 热更新需要的插件
        new webpack.HotModuleReplacementPlugin(),
    ],
    output: {
        path: path.resolve(__dirname, "dist"),// webpack后的bundle.js所在目录
        filename: "bundle.js"
    },
    devServer: { // 开发服务器配置
        contentBase: "./dist", // 起服务器的目录,指定了服务器资源的根目录,服务器打包的bundle.js在内存中，通过改路径引用，而非output路径
        historyApiFallback: true,
        port: 8000,
        hot: true,
        inline: true
    }
}

```
划重点，要使用webpack-dev-server需要配置以下内容
* 一个插件：
```bash
 plugins: [ // 热更新需要的插件
        new webpack.HotModuleReplacementPlugin(),
    ],
```
* 服务器配置文件
```bash
 devServer: { // 开发服务器配置
        contentBase: "./", // 起服务器的目录,指定了服务器资源的根目录,经过服务器打包的bundle.js在内存中，通过该路径引用，而非webpack打包的output路径
        historyApiFallback: true,
        port: 8000,  // 监听8000端口
        hot: true, // 是否热更新
        inline: true // inline模式
    }
```
后面几个参数看文档很好理解,重点理解第一个参数：contentBase，该参数规定了你的服务器根目录，比如apache的www文件夹，通过命令行参数将服务器运行起来，会自动打包生成一个bundle.js，但是这个bundle.js并不是我们原来使用webpack命令生成的那个bundle.js，可能这样说有点混乱。
以下是我的理解：
* 命令行使用webpack命令，会根据webpack.config.js配置文件直接进行打包，并输出至output路径；
* 通过命令行启动webpack-dev-server服务，也会根据webpack.config.js配置文件直接进行打包，但不会在output路径下生成一个bundle.js，而是在内存中生成，每次文件发生变化也会自动生成,而是在服务器根目录contentBase下存在一个bundle.js，我们看不见。

比如文件结构如下：
```bash
----index.html
----dist
--------bundle.js
```
我们会在index.html中如此引用bundle.js
```js
<script type="text/javascript" src="./dist/bundle.js"></script>
```
没毛病。
但是如果你起了一个webpack-dev-server，服务器根目录为'./'即项目根目录，那么这样你的网页并不会实时刷新，因为引用的一直是webpack打包生成的那个bundle.js，而非webpack-dev-server生成的bundle.js，我们应当改为：
```js
<script type="text/javascript" src="./bundle.js"></script>
```
这样便可以引用到最新的bundle.js，实现实时刷新。

其实可以将index.html也放进dist目录，再将contentBase更改为'./dist'

如此一来webpack生成的bundle.js与webpack-dev-server生成的bundle.js便可以通过相同路径引用了，

```js
<script type="text/javascript" src="./bundle.js"></script>
```

webpack-dev-server的原理我猜应该是通过检测文件变化，随后使用websocket之类的能够由服务器向客户端发送信息的方式进行实时刷新。

对了，在package.json中配置如下命令可以通过npm start快速执行webpack-dev-server，少打几个字母。
```js
"scripts": {
    "start": "webpack-dev-server --inline"
  },
```
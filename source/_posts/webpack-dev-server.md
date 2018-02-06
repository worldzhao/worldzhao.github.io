---
title: webpack之webpack-dev-server
date: 2017-10-02
tags: [webpack]
categories: 构建工具
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
* 使用file-loader以及url-loader对图片进行处）。
代码如下：
```js
var webpack = require('webpack');
var path = require('path');

module.exports = {
    entry: {
        // path.join用于连接路径。该方法的主要用途在于，会正确使用当前系统的路径分隔符，Unix系统是"/"，Windows系统是"\"。
        // 同时__dirname是指当前文件目录的绝对路径，我们一般写绝对路径避免出现一些问题
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
    plugins: [ // 热更新插件
        new webpack.HotModuleReplacementPlugin(),
    ],
    output: {
        path: path.resolve(__dirname, "dist"),// webpack后的bundle.js所在目录
        filename: "bundle.js",
        publicPath: '/public/'
    },
    devServer: { // 开发服务器配置
        // '0,0,0,0'表示我们可以用任何方式进行访问 如localhost 127.0.0.1 以及外网ip 若配置为'localhost'
        // 或'127.0.0.1'别人无法从外网进行调试
        host: '0.0.0.0',
        // 起服务的端口
        port: '8888',
        // 起服务的目录，此处与output一致，此处有坑(要考虑publicPath，还要知晓webpack-dev-server生成的文件在内存中，若项目中已经有
        // 了dist则以项目中的dist为准，所以删掉dist吧)
        contentBase: './',
        // 热更新 如果不配置webpack-dev-server会在文件修改后全局刷新而非局部替换
        hot: true,
        overlay: {
        // 如果打包过程中出现错误在浏览器中渲染一层overlay进行展示
        errors: true
        },
        // 以下两项弄懂了publicPath就好理解
        // 在此处设置与output相同的publicPath,把静态资源文件放在public文件夹下
        // 使得output.publicPath得以正常运行，其实这里的publicPath更像是output.path
        publicPath: '/public/',
        // 解决刷新404问题（服务端没有前端路由指向的文件） 全都返回index.html
        historyApiFallback: {
        index: '/public/index.html'
        }
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

最好的办法是将webpack-dev-server的contentBase与webpack.config.js的output.path设为一致，如此一来webpack生成的bundle.js与webpack-dev-server生成的bundle.js便可以通过相同路径引用了,即
```js
contentBase: path.resolve(__dirname, "dist")
```
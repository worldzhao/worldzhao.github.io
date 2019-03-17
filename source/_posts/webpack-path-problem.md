---
title: 【转】webpack之图片打包及路径问题
date: 2017-10-02
tags: [webpack]
categories: 工程化
---

> 转自[webpack 踩坑之路 (2)——图片的路径与打包](http://www.cnblogs.com/ghost-xyx/p/5812902.html)

## 正文

在实际生产中有以下几种图片的引用方式：

1.  HTML 文件中 img 标签的 src 属性引用或者内嵌样式引用

```
<img src="imgPath" />
<div style="background:url(imgPath)"></div>
```

2.  CSS 文件中的背景图等设置

```
.photo { background: url(imgPath);}
```

3.  JavaScript 文件中动态添加或者改变的图片引用

```
var imgTempl = `<img src="imgPath" />`;
document.body.innerHTML = imgTempl;
```

4.  ReactJS 中图片的引用

```
import React from 'react';
import ReactDOM from 'react-dom';

class App extends React.Component {
    render() {
        return (<img src='imgPath' />);
    }
}

ReactDom.render(<App/>, document.querySelector('#container'));
```

**url-loader**

在 webpack 中引入图片需要依赖 url-loader 这个加载器。

安装：

```bash
npm install url-loader --save-dev
```

当然你可以将其写入配置中，以后与其他工具模块一起安装。

在 webpack.config.js 文件中配置如下：

```bash
module: {
    loaders: [
        {
            test: /\.(png|jpg)$/,
            loader: 'url-loader?limit=8192'
        }
    ]
}
```

test 属性代表可以匹配的图片类型，除了 png、jpg 之外也可以添加 gif 等，以竖线隔开即开。

loader 后面  limit  字段代表图片打包限制，这个限制并不是说超过了就不能打包，而是指当图片大小小于限制时会自动转成 base64 码引用。上例中大于 8192 字节的图片正常打包，小于 8192 字节的图片以 base64 的方式引用。

url-loader 后面除了 limit 字段，还可以通过 name 字段来指定图片打包的目录与文件名：

```
module: {
    loaders: [
        {
            test: /\.(png|jpg)$/,
            loader: 'url-loader?limit=8192&name=images/[hash:8].[name].[ext]'
        }
    ]
}
```

上例中的 name 字段指定了在打包根目录（output.path）下生成名为 images 的文件夹，并在原图片名前加上 8 位 hash 值。
例：工程目录如下:

![project.png](http://upload-images.jianshu.io/upload_images/4869616-34f583ff266893e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 main.css 中引用了同级 images 文件夹下的 bg.jpg 图片

```css
background-image: url(./images/bg.jpg);
```

通过之前的配置，使用 \$ webpack 命令对代码进行打包后生成如下目录:

![images.png](http://upload-images.jianshu.io/upload_images/4869616-74d9ee7e759d3d4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打包目录中，css 文件和 images 文件夹保持了同样的层级，可以不做任务修改即能访问到图片。区别是打包后的图片加了 hash 值，bundle.css 文件里引入的也是有 hash 值的图片。

```
background-image: url(images/f593fbb9.bg.jpg);
```

（上例中，使用了单独打包 css 的技术，只是为了方便演示）

**publicPath**

output.publicPath 表示资源的发布地址，当配置过该属性后，打包文件中所有通过**相对路径**引用的资源都会被配置的路径所替换。

```
output: {
  path: 'dist',
  publicPath:  '/assets/',
  filename: 'bundle.js'
}
```

main.css

```css
background-image: url(./images/bg.jpg);
```

bundle.css

```css
background-image: url(/assets/images/f593fbb9.bg.jpg);
```

该属性的好处在于当你配置了图片 CDN 的地址，本地开发时引用本地的图片资源，上线打包时就将资源全部指向 CDN 了。
但是要注意，如果没有确定的发布地址不建议配置该属性，否则会让你打包后的资源路径很混乱。

**JS 中的图片**

初用 webpack 进行项目开发的同学会发现：在 js 或者 react 中引用的图片都没有打包进 bundle 文件夹中。
正确写法应该是通过模块化的方式引用图片路径，这样引用的图片就可以成功打包进 bundle 文件夹里了

**js**

```
var imgUrl = require('./images/bg.jpg'),
    imgTempl = '<img src="'+imgUrl+'" />';
document.body.innerHTML = imgTempl;
```

**react**

```
render() { return (<img src={require('./images/bg.jpg')} />);}
```

**HTML 中的图片**

由于 webpack 对 html 的处理不太好，打包 HTML 文件中的图片资源是相对来说最麻烦的。这里需要引用一个插件—— html-withimg-loder

```bash
$ npm install html-withimg-loader --save-dev
```

webpack.config.js 添加配置

```
module: {
    loaders: [
        {
            test: /\.html$/,
            loader: 'html-withimg-loader'
        }
    ]
}
```

在 bundle.js 中引用 html 文件

```
import '../index.html';
```

这样 html 文件中的图片就可以被打包进 bundle 文件夹里了。

**感谢您的浏览，希望能有所帮助**

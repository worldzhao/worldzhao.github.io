---
title: Nginx学习笔记-项目部署
date: 2018-08-18
tags: [nginx]
categories: 运维知识
---

# nginx 学习笔记-项目部署

当项目在本地开发完毕后，我们需要将项目部署到服务器上

```zsh
# nginx目录
/data/www

# nginx配置文件位置
/etc/nginx/nginx.conf
```

## 通过 ssh 登录 centos 服务器

关键：ssh 相关知识

## 在本地通过 scp 将压缩后的打包文件上传至服务器

关键：scp/压缩/解压缩等命令

## 进行配置 nginx

现在我们已经将压缩文件解压，并将里面的内容移至了`/data/www`根目录，直接访问服务器地址[默认为 80 端口]应当是直接可以访问到 html 文件的，这已经很好了，我们的前端项目已经可以被别人访问，但是还有一些问题：

1.  单页面应用刷新报 404 错误
2.  如何将接口代理至其他服务

### 解决刷新 404 问题

`react-router`和`vue-router`是我们常用的前端路由库，监听 url 的变化刷新视图，注意这一过程和传统的多页应用不一样，是没有经过服务端的，也就是说如果进行刷新，浏览器请求服务端的相应路径，但是我们知道服务端根本没有对这个路径进行任何处理，即服务端该路径不存在任何文件也没有代理到别处，所以就产生了 404 问题。

解决方法：代理到`index.html`，重新通过前端路由进行渲染

实际配置如下：

```
location / {
    try_files $uri /index.html;
    index index.html;
}
```

如此一来，我们所有的请求都会直接定位到`index.html`，由 js 接手页面的渲染。

### api 接口转发

目前通过 nginx 监听 80 端口，并部署了前端项目。

在服务器 3000 端口上启动了 api 服务，在前端项目中通过`/api/personalized`的形式进行请求，我们知道这种方式默认是请求`http//your-server/api/personalized`的，很明显，80 端口上并没有该服务，我们需要监听来自 80 端口以`/api/`开头的请求，将其转到服务器本机的 3000 端口，即`/api/personalized`实际请求的地址为`http:your-server:3000/personalized`，注意，我们通过添加了`/api/`前缀来区分接口，所以转发时还需要移除该前缀。

实际配置如下：

```
location /api/ {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass   http://localhost:3000;
}
```

---
title: Nginx学习笔记-基本操作
date: 2018-07-08
tags: [nginx]
categories: 运维知识
---

**转自腾讯云实验室**

搭建 Http 静态服务器环境
搭建静态网站，首先需要部署环境。

下面的步骤，将告诉大家如何在服务器上通过 Nginx 部署 HTTP 静态服务。

## 安装 Nginx

在 CentOS 上，可直接使用 yum 来安装 Nginx
yum install nginx -y
安装完成后，使用 nginx 命令启动 Nginx：

```
nginx
```

此时，访问 http://119.29.241.71 可以看到 Nginx 的测试页面

## 配置静态服务器访问路径

外网用户访问服务器的 Web 服务由 Nginx 提供，Nginx 需要配置静态资源的路径信息才能通过 url 正确访问到服务器上的静态资源。

打开 Nginx 的默认配置文件 `/etc/nginx/nginx.conf` ，修改 Nginx 配置，将默认的 `root /usr/share/nginx/html` 修改为: `root /data/www`

配置文件将 `/data/www/static` 作为所有静态资源请求的根路径，如访问: `http://119.29.241.71/static/index.js`，将会去 `/data/www/static/` 目录下去查找 index.js。现在我们需要重启 Nginx 让新的配置生效，如：

```
nginx -s reload
```

重启后，现在我们应该已经可以使用我们的静态服务器了，现在让我们新建一个静态文件，查看服务是否运行正常。
首先让我们在 `/data` 目录 下创建 www 目录，如：

```
mkdir -p /data/www
```

## 创建第一个静态文件

在 `/data/www` 目录下创建我们的第一个静态文件 index.html
现在访问 http://119.29.241.71/index.html 应该可以看到页面输出 index.html 中的内容

到此，一个基于 Nginx 的静态服务器就搭建完成了，现在所有放在 /data/www 目录下的的静态资源都可以直接通过域名访问。

### `nginx -s reload`报错

如下报错信息

```
nginx: [error] open() "/var/run/nginx.pid" failed (2: No such file or directory)
```

执行

```
# sudo nginx -c /etc/nginx/nginx.conf
```

然后再执行

```
nginx -s reload
```

## 总结

| 命令                           | 解释                                       |
| ------------------------------ | ------------------------------------------ |
| nginx                          | 启动 nginx 服务                            |
| nginx -s reload                | 重载 nginx 服务                            |
| nginx -c /etc/nginx/nginx.conf | 设置配置文件 (默认: /etc/nginx/nginx.conf) |

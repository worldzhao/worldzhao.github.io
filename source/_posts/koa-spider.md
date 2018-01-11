---
title: koa-spider
date: 2017-07-21
tags: [koa]
categories: node
---
### koa-spider
>[github地址](https://github.com/hackerwen/koa-spider)
#### 简述
通过koa2搭建服务器。定时爬取github获取星数最多的前九名信息存入本地json文件。当服务器接受请求时返回json.

#### 使用方法
克隆项目至本地后进入项目根目录

    npm install//安装依赖

    cd spider

    node index.js //启动爬虫

    cd..

    node app.js//启动服务器

    在浏览器访问"local:3000/github/star/java" 
    语言选项：['javascript', 'html', 'css', 'java', 'python', 'php', 'ruby', 'typescript', 'coffeescript']

#### 依赖的库

1. koa koa-router搭建服务器
2. request/request-promise，使用async/await+promise控制异步流程更为方便
3. cheerio服务端jquery，进行Dom解析
4. node-schedule控制爬虫定时爬取的一个库
5. mkdirp创建存放json的文件夹
6. koa2-cors跨域处理

#### 返回数据展示

    localhost:3000/github/star/python

    {
    "data": [
        {
            "name": "tensorflowtensorflow",
            "star": "64.2k",
            "des": "Computation using data flow graphs for scalable machine learning",
            "update": "Jul 21, 2017"
        },
        {
            "name": "robbyrusselloh-my-zsh",
            "star": "56.4k",
            "des": "A delightful community-driven (with 1,000+ contributors) framework for managing your zsh configuration. Includes 200+…",
            "update": "Jul 20, 2017"
        },
        {
            "name": "vintaawesome-python",
            "star": "36.3k",
            "des": "A curated list of awesome Python frameworks, libraries, software and resources",
            "update": "Jul 19, 2017"
        },
        {
            "name": "jakubroztocilhttpie",
            "star": "30.5k",
            "des": "Modern command line HTTP client – user-friendly curl alternative with intuitive UI, JSON support, syntax highlighting…",
            "update": "Jul 20, 2017"
        },
        {
            "name": "nvbnthefuck",
            "star": "29.1k",
            "des": "Magnificent app which corrects your previous console command.",
            "update": "Jul 18, 2017"
        },
        {
            "name": "palletsflask",
            "star": "28.4k",
            "des": "A microframework based on Werkzeug, Jinja2 and good intentions",
            "update": "Jul 20, 2017"
        },
        {
            "name": "blueimpjQuery-File-Upload",
            "star": "27.1k",
            "des": "File Upload widget with multiple file selection, drag&drop support, progress bar, validation and preview images, audi…",
            "update": "Jul 19, 2017"
        },
        {
            "name": "djangodjango",
            "star": "27k",
            "des": "The Web framework for perfectionists with deadlines.",
            "update": "Jul 21, 2017"
        },
        {
            "name": "requestsrequests",
            "star": "26.3k",
            "des": "Python HTTP Requests for Humans™ ✨🍰✨",
            "update": "Jul 20, 2017"
        },
        {
            "name": "ansibleansible",
            "star": "24.3k",
            "des": "Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy. Avoid…",
            "update": "Jul 21, 2017"
        }
    ]
    }

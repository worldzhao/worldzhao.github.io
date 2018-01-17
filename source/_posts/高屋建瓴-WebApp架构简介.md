---
title: 高屋建瓴-WebApp架构简介
date: 2018-01-15
tags: [技术思考]
categories: 高屋建瓴
---
# WebApp架构简介

> 该文章总结自：[慕课网: Webpack+React全栈工程架构项目实战精讲](https://coding.imooc.com/class/161.html)

## 工程架构

1. 解放生产力。浏览器实时刷新/打包压缩/项目环境构建等等重复性工作无需重复解决，解放开发人员生产力：
    * 源代码预处理；
    * 自动打包，自动更新页面显示；
    * 自动处理图片依赖，保证开发和正式环境的统一。

2. 围绕解决方案搭建环境。根据不同的解决方案搭建不同的环境，使用React有针对jsx以及其生态圈的一套环境，使用Angular有针对typescript以及angular生态圈的一套环境等等：
    * 不同的前端框架需要不同的运行架构；
    * 预期可能出现的问题并规避。例如项目初期编写css较为随意，后期项目扩大，类名冲突等问题接踵而来，要不要使用CSS Module或者预处理器等。

3. 保证项目质量。代码规范检查eslint的使用等等：
    * code lint；
    * 不同环境排除差异 .editorconfig文件 indent_style/indet_size/charset/end_of_line/insert_final_newline/trim_trailing_whitespace
    * git commit 预处理

关键字： **定制**

## 项目架构

与工程架构不同的是，项目架构更多的考虑是网页如何去运行，业务代码如何去分层，将来如何更好地扩展。

1. 技术选型     [vue/react/angular]
2. 数据解决方案 [redux/reflux/mobx]
3. 整体代码风格 [目录结构/命名方式/数据存储redux|mobx|component]


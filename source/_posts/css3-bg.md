---
title: css3背景
date: 2017-03-20
tags: [css3]
categories: css
---
### 背景图片
之前的背景图片在容器中只能显示容器大小的部分.
css3在css2基础上加入了更为灵活的背景图片机制,可以十分容易的控制背景图片.而不用分散精力在背景图的大小上.

背景大小:
* background-size:水平宽度 垂直高度.

* background-size:cover;完全覆盖盒子,可能会超出盒子.

* background-size:contain;保证背景图片最大化的在盒子中**等比例**显示但是不保证能铺满盒子.

<!-- more -->

这两个属性可以很好的使得图片自适应容器大小,对响应式设计十分友好.

背景原点:控制背景从什么地方开始显示
background-origin:padding-box(默认)/border-box/content-box

背景裁剪:与背景原点搭配使用,裁剪背景图片.
background-clip属性值为border-box/padding-box/content-box
当超出背景裁剪属性值所代表的box时,超出部分clip掉,例如背景原点在padding-box但是背景裁剪在content-box,那么padding-box及content-box之间的部分会被clip.

多背景:给盒子加多个背景,按照背景语法格式书写
		background: url('images/bg1.png') no-repeat left top
		,url('images/bg2.png') no-repeat right top
		,url('images/bg3.png') no-repeat right bottom
		,url('images/bg4.png') no-repeat left bottom;

background-position:确定背景位置 水平位置 垂直位置left top左上
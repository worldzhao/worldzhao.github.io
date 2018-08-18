---
title: 盒子模型与box-sizing
date: 2017-03-18
tags: [盒子模型]
categories: css
---

# 传统的盒子模型

最终表现出来的宽度=内容宽度(width)+2padding+2border

最终表现出来的高度=内容高度(height)+2padding+2border

如果改变了 padding,盒子整个展现出来的大小就会发生变化,即使我们设置的 width 是个定值并没有变化.

实际上存在三个盒子:

1.  content-box 实际内容盒子 content-box

2.  padding-box content-box 包上 padding 的 padding-box

3.  border-box padding-box 加上 border 的 border-box

此处引入 box-sizing 属性,该属性有两个值

1.  content-box(默认,外加模式)

2.  border-box(内减模式).

box-sizing 后面的属性值所代表的盒子宽度是始终不会变的,始终为我们设置的 width.

默认是 content-box,即我们的 width 设置的是 content-box 的宽度.所以加上 padding 和 border 实际表现出来的宽度会变大,这种模式被称为外加模式.

如果修改为 border-box,设置的 width 就是最终表现出来的宽度,因为 border-box 的宽度就是我们设置的 width 值,如果 padding 值也确定了,那么只会减少 content-box 的宽度,这种模式被成为内减模式.

有了 box-sizing:border-box; 以后给盒子动态增加 border padding 不必担心会撑大盒子造成溢出换行等混乱

注: 上述讨论的为 W3C 盒子模型（1）盒子模型有两种， IE 盒子模型、W3C 盒子模型；（2）盒模型： 内容(content)、填充(padding)、边界(margin)、 边框(border)；（3）区 别： IE 的 content 部分把 border 和 padding 计算了进去;

很明显 IE 的盒子模型更符合人的认知,于是 W3C 才有了后续的 box-sizing.

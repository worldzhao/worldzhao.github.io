---
title: 盒子模型与box-sizing
date: 2017-03-18
tags: [css3]
categories: css
---
# 传统的盒子模型
最终表现出来的宽度=内容宽度(width)+2padding+2border  

最终表现出来的高度=内容高度(height)+2padding+2border  

如果改变了padding,盒子整个展现出来的大小就会发生变化,即使我们设置的width是个定值并没有变化.  

实际上存在三个盒子:  

1. content-box  实际内容盒子 content-box
 
2. padding-box  content-box包上padding的padding-box  

3. border-box   padding-box加上border的border-box

此处引入box-sizing属性,该属性有两个值
1. content-box(默认,外加模式)  

2. border-box(内减模式).  


box-sizing后面的属性值所代表的盒子宽度是始终不会变的,始终为我们设置的width.  

默认是content-box,即我们的width设置的是content-box的宽度.所以加上padding和border实际表现出来的宽度会变大,这种模式被称为外加模式.  

如果修改为border-box,设置的width就是最终表现出来的宽度,因为border-box的宽度就是我们设置的width值,如果padding值也确定了,那么只会减少content-box的宽度,这种模式被成为内减模式.

有了box-sizing:border-box; 以后给盒子动态增加border padding不必担心会撑大盒子造成溢出换行等混乱

注:  上述讨论的为W3C盒子模型
（1）盒子模型有两种， IE 盒子模型、W3C 盒子模型；
（2）盒模型： 内容(content)、填充(padding)、边界(margin)、 边框(border)；
（3）区  别： IE的content部分把 border 和 padding计算了进去;

很明显IE的盒子模型更符合人的认知,于是W3C才有了后续的box-sizing.
---
title: css3文本效果
date: 2017-03-20
tags: [css3]
categories: css
---
### 新增颜色模式-rgba hsla
## rgba
是什么？
* r:红色 g:绿色 b:蓝色 0-255
* a:透明度0-1

能做什么？
*  盒子内背景透明，文字不透明，给浅色系文字添加深色系透明背景。不影响背景图片的显示，同时突出文字。   

css2:
opacity的局限性，给父盒子设置透明度，子盒子受到影响，且子盒子无法改变透明度。
transparent完全透明，无法改变透明度。
而rgba是颜色，可以做用与背景，直接改变背景色的透明度。

## hsla
* h:色调 0-360 s:饱和度 0-100% l:亮度 0-100% a:透明度  


<!-- more -->

### 文字阴影-text-shadow
* 4个参数
1. x偏移量(+右-左) 
2. y偏移量(+下-上)  
3. 模糊程度 
4. 阴影颜色 


    text-shadow:0 0 10px red;

* 阴影叠加


    text-shadow:0 0 10px red,-5px 10px 10px blue,....;

* 浮雕效果


    p{
        font-size: 100px;
        line-height: 200px;
        color: #fff;
        text-shadow: 2px 2px  4px  #000;
    }


* 凹凸文字效果   

    .p1{//凸
        text-shadow:-1px -1px  0 white,1px 1px 0 black;

    }
    .p2{//凹
        text-shadow:-1px -1px  0 black,1px 1px 0 white;
    }
  

* 文字模糊

1. 将自身颜色调整为黑色完全透明
2. 用大阴影将其遮住(模糊程度大)
3. 将阴影颜色调整为黑色透明度0.5，即可生成文字模糊效果。


    h1{
        color: rgba(0,0,0,0);
        text-shadow: 0 0 5px rgba(0,0,0,0.5);
    }

### 文字描边 -webkit-text-stroke:宽度 颜色
### 文字排版 direction+unicode-bidi:bidi-overrider;
右对齐并且文字从右向左(**全兼容**)

    p{
        width: 200px;
        height: 50px;
        border: 1px solid red;
        direction: rtl;
        unicode-bidi:  bidi-override;
    }

### 文字超出部分显示省略号 text-overflow

    p{
        width: 500px;
        overflow: hidden;
        white-space: nowrap;/*不准换行*/
        -ms-text-overflow: ellipsis;
        text-overflow: ellipsis;/*显示省略号*/
     }
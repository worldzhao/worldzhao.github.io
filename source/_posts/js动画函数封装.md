---
title: js动画函数封装
date: 2017-03-16
tags: [js]
categories: js
---
* 前言
这是我的第一篇关于js的文章，如有错误，恳请指正。
这一系列文章的主题是复习动画函数，本文为第一篇。
动画并不是真的在动，而是每隔一个极小的时间将一个element移动一个极小的距离，使人产生动画的效果，如果读者看过小时候快速翻页就会产生动画效果的小人书那应当不难理解。

<!-- more -->

好了，让我们开始吧。

* 函数最终代码（初步）

//elem:要移动的元素 fianl_x final_y 目的地坐标(相对于父盒子)
```js
function animate(elem, final_x, final_y) {
  clearInterval(elem.timer); //要用定时器，先清定时器
  elem.timer = setInterval(function () {
    var x = parseInt(elem.style.left) || 0;
    var y = parseInt(elem.style.top) || 0;
    if (x === final_x && y === final_y) {
      clearInterval(elem.timer); //用完定时器，清除定时器
      return true;
    }
    var distx = (final_x - x) / 10;
    distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx);
    x = x + distx;
    var disty = (final_y - y) / 10;
    disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty);
    y = y + disty;
    elem.style.left = x + "px";
    elem.style.top = y + "px";
  }, 25);
}
```

1. 准备工作
首先，要移动的元素必须是绝对定位的，前面也说了，我们是不断地在一个极小的事件不断的更新元素的横竖位置(相对于父盒子)，而符合我们要求的便是绝对定位了，通过更改left,right,top,bottom等属性的值来更改元素的位置。参数里有元素的最终位置，我们也需要获取元素的初始位置。
注意:此处要转为整型，因为ele.style.left获取的是一个字符串，带单位"px".

```js
function animate(elem,final_x,fianl_y){
    var x,y;
    if(elem.style.left){
         x = parseInt(elem.style.left);
    }else{
         x = 0;
    }
    if(elem.style.top){
         y = parseInt(elem.style.top);
    }else{
         y = 0;
    }
}
```
		

但是可能这个元素并没有设置left以及top的值，于是我们需要进行检测,如果没有这两个值则将它们设置为0(实际上一个绝对定位的元素如果没有设置值的话默认为0，0)；

好啦，现在我们得到了元素的起始位置，那么什么时候这个函数才算完成了任务呢？没错，就是起始位置与终止位置相等时。于是我们加入函数结束的条件。

```js
if(x==final_x&&y==final_y){//如果起始位置与目标位置一致
    return true;
}
```

接下来我们的问题就是该怎么移动这个元素了。  

不断的移动一个微小的距离，造成动画的效果。  

敲黑板，划重点。    

“不断”。没错就是我们的setInterval(a,b)定时器。这个函数有两个参数，第一个参数a是要执行的代码块，第二个参数b是执行的间隔时间,这里我们让它每25ms执行一次吧。  

我们的逻辑就是：每隔一段时间，获取一次元素当前位置，让元素移动一个微小的距离，如果当前位置等于目标位置，我们就退出setInterval函数,如果不等于，则重复执行代码块a。

现在的代码如下：  

```js
setInterval(function(){
    var x,y;
    if(elem.style.left){
         x = parseInt(elem.style.left);
    }else{
         x = 0;
    }
    if(elem.style.top){
         y = parseInt(elem.style.top);
    }else{
         y = 0;
    }
    if(x==final_x&&y==final_y){
        return true;
    }
},25);
```
    

"一个微小的距离"。即我们每个25ms让其移动的距离，这里我们取10px;

```js
setInterval(function () {
  var x, y;
  if (elem.style.left) {
    x = parseInt(elem.style.left);
  } else {
    x = 0;
  }
  if (elem.style.top) {
    y = parseInt(elem.style.top);
  } else {
    y = 0;
  }
  if (x === final_x && y === final_y) {
    return true;
  }
  var dist = 10; //我们移动的距离
  if (x < fianl_x) { //如果横向在目标位置左侧，则+10向目标位置前进
    x = x + 10;
  }
  if (x > final_x) { //如果横向在目标位置右侧，则-10向目标位置后退
    x = x - 10;
  }
  if (y < fianl_y) { //同上
    y = y + 10;
  }
  if (y > fianl_y) {
    y = y - 10;
  }
}, 25);
```

然后再将增加或减少了的x,y重新赋值给元素的left以及top属性

```js
elem.style.left = x + "px";
elem.style.top = y + "px";
```

这样，我们就初步完成了动画函数，每隔25ms将元素移动10px(x轴与y轴)，如果当前位置等于了目标位置，则退出setInterval函数，animate执行完毕。此时代码清单如下:
```js
function animate(elem, final_x, fianl_y) {
  setInterval(function () {
    var x, y;
    if (elem.style.left) {
      x = parseInt(elem.style.left);
    } else {
      x = 0;
    }
    if (elem.style.top) {
      y = parseInt(elem.style.top);
    } else {
      y = 0;
    }
    if (x == final_x && y == final_y) {
      return true;
    }
    var dist = 10; //我们移动的距离
    if (x < fianl_x) { //如果横向在目标位置左侧，则+10向目标位置前进
      x = x + 10;
    }
    if (x > final_x) { //如果横向在目标位置右侧，则-10向目标位置后退
      x = x - 10;
    }
    if (y < fianl_y) { //同上
      y = y + 10;
    }
    if (y > fianl_y) {
      y = y - 10;
    }
  }, 25);
  elem.style.left = x + "px";
  elem.style.top = y + "px";
}
```
	

但是这串代码既不健壮也不简洁，还可以优化。  

我们可以先简化一下代码：  
```js
function animate(elem, final_x, final_y) {
  setInterval(function () {
    var x = parseInt(elem.style.left) || 0;
    var y = parseInt(elem.style.top) || 0;
    if (x === final_x && y === final_y) {
      return true;
    }
    var dist = 10;
    if (x < fianl_x) { //如果横向在目标位置左侧，则+10向目标位置前进
      x += 10;
    }
    if (x > final_x) { //如果横向在目标位置右侧，则-10向目标位置后退
      x -= 10;
    }
    if (y < fianl_y) { //同上
      y += 10;
    }
    if (y > fianl_y) {
      y -= 10;
    }
    elem.style.left = x + "px";
    elem.style.top = y + "px";
  }, 25);
}
```
    

问题1. 移动距离每次都是固定的(10px)，是匀速移动。我们想逼真一点，当距离目标位置越近时速度变得越慢，产生渐变的效果；  

问题2. 定时器没有及时清除。  

假如你先想通过调用animate函数将elem移动到目标位置1，在其还未到达的时候，又改变主意让它移动到目标位置2，再调用animate函数，传入了新的final_x,final_y，此时就会造成两个定时器同时运作，产生混乱。  

定时器A想让它到目标位置1，定时器B想让它到目标位置2,获取的元素当前位置却是同一个，就会造成elem摇摆不定，所以要以后一次的调用为准，清除前一次的定时器。对了，到达目标位置后也要清除，总不能把人家白白搁在那。  

要养成好习惯：要用定时器，先清定时器。用完定时器，清除定时器。

首先解决问题1。

```js
var distx = (final_x - x) / 10;
var disty = (final_y - y) / 10;
```
	
此时，我们的移动的距离不再是相同的10px，而是根据目标位置与当前位置的差（距离差）来进行动态计算，当前位置距离目标位置越近我们移动的距离越小,每次移动距离差的十分之一，这样我们就不再是匀速运动了，距离差在不断减小，我们移动距离dist也在不断减小。
那些if也让人看着生厌，这时我们的distx,disty已经带有正负号，可以直接加在x,y上了。   
/10要有取整处理，正负数的向上取整不同。代码如下：  

```js
var distx = (final_x - x) / 10;
distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx);
x = x + distx;
var disty = (final_y - y) / 10;
disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty);
y = y + disty;
```
   

问题1结束。

其次解决问题2。

我们可以知道，多个定时器都是作用在同一个元素上面，我们可以将每次进行的定时器的ID绑定在元素上，下一次执行新的定时器时将元素本身的定时器clear（也就是上一个定时器）,再将此次定时器ID绑定即可。

最终代码：

```js
function animate(elem, final_x, final_y) {
  clearInterval(elem.timer); //要用定时器，先清定时器
  elem.timer = setInterval(function () {
    var x = parseInt(elem.style.left) || 0;
    var y = parseInt(elem.style.top) || 0;
    if (x === final_x && y === final_y) {
      clearInterval(elem.timer); //用完定时器，清除定时器
      return true;
    }
    var distx = (final_x - x) / 10;
    distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx);
    x = x + distx;
    var disty = (final_y - y) / 10;
    disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty);
    y = y + disty;
    elem.style.left = x + "px";
    elem.style.top = y + "px";
  }, 25);
}
```


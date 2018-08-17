---
title: js动画函数封装
date: 2017-03-16
tags: [js动画]
categories: js
---

- 前言
  动画并不是真的在动，而是每隔一个极小的时间将一个 element 移动一个极小的距离，使人产生动画的效果，如果读者看过小时候快速翻页就会产生动画效果的小人书那应当不难理解。

<!-- more -->

好了，让我们开始吧。

- 函数最终代码（初步）

//elem:要移动的元素 fianl_x final_y 目的地坐标(相对于父盒子)

```js
function animate(elem, final_x, final_y) {
  clearInterval(elem.timer) //要用定时器，先清定时器
  elem.timer = setInterval(function() {
    var x = parseInt(elem.style.left) || 0
    var y = parseInt(elem.style.top) || 0
    if (x === final_x && y === final_y) {
      clearInterval(elem.timer) //用完定时器，清除定时器
      return true
    }
    var distx = (final_x - x) / 10
    distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx)
    x = x + distx
    var disty = (final_y - y) / 10
    disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty)
    y = y + disty
    elem.style.left = x + 'px'
    elem.style.top = y + 'px'
  }, 25)
}
```

1.  准备工作
    首先，要移动的元素必须是绝对定位的，前面也说了，我们是不断地在一个极小的事件不断的更新元素的横竖位置(相对于父盒子)，而符合我们要求的便是绝对定位了，通过更改 left,right,top,bottom 等属性的值来更改元素的位置。参数里有元素的最终位置，我们也需要获取元素的初始位置。
    注意:此处要转为整型，因为 ele.style.left 获取的是一个字符串，带单位"px".

```js
function animate(elem, final_x, fianl_y) {
  var x, y
  if (elem.style.left) {
    x = parseInt(elem.style.left)
  } else {
    x = 0
  }
  if (elem.style.top) {
    y = parseInt(elem.style.top)
  } else {
    y = 0
  }
}
```

但是可能这个元素并没有设置 left 以及 top 的值，于是我们需要进行检测,如果没有这两个值则将它们设置为 0(实际上一个绝对定位的元素如果没有设置值的话默认为 0，0)；

好啦，现在我们得到了元素的起始位置，那么什么时候这个函数才算完成了任务呢？没错，就是起始位置与终止位置相等时。于是我们加入函数结束的条件。

```js
if (x == final_x && y == final_y) {
  //如果起始位置与目标位置一致
  return true
}
```

接下来我们的问题就是该怎么移动这个元素了。

不断的移动一个微小的距离，造成动画的效果。

敲黑板，划重点。

“不断”。没错就是我们的 setInterval(a,b)定时器。这个函数有两个参数，第一个参数 a 是要执行的代码块，第二个参数 b 是执行的间隔时间,这里我们让它每 25ms 执行一次吧。

我们的逻辑就是：每隔一段时间，获取一次元素当前位置，让元素移动一个微小的距离，如果当前位置等于目标位置，我们就退出 setInterval 函数,如果不等于，则重复执行代码块 a。

现在的代码如下：

```js
setInterval(function() {
  var x, y
  if (elem.style.left) {
    x = parseInt(elem.style.left)
  } else {
    x = 0
  }
  if (elem.style.top) {
    y = parseInt(elem.style.top)
  } else {
    y = 0
  }
  if (x == final_x && y == final_y) {
    return true
  }
}, 25)
```

"一个微小的距离"。即我们每个 25ms 让其移动的距离，这里我们取 10px;

```js
setInterval(function() {
  var x, y
  if (elem.style.left) {
    x = parseInt(elem.style.left)
  } else {
    x = 0
  }
  if (elem.style.top) {
    y = parseInt(elem.style.top)
  } else {
    y = 0
  }
  if (x === final_x && y === final_y) {
    return true
  }
  var dist = 10 //我们移动的距离
  if (x < fianl_x) {
    //如果横向在目标位置左侧，则+10向目标位置前进
    x = x + 10
  }
  if (x > final_x) {
    //如果横向在目标位置右侧，则-10向目标位置后退
    x = x - 10
  }
  if (y < fianl_y) {
    //同上
    y = y + 10
  }
  if (y > fianl_y) {
    y = y - 10
  }
}, 25)
```

然后再将增加或减少了的 x,y 重新赋值给元素的 left 以及 top 属性

```js
elem.style.left = x + 'px'
elem.style.top = y + 'px'
```

这样，我们就初步完成了动画函数，每隔 25ms 将元素移动 10px(x 轴与 y 轴)，如果当前位置等于了目标位置，则退出 setInterval 函数，animate 执行完毕。此时代码清单如下:

```js
function animate(elem, final_x, fianl_y) {
  setInterval(function() {
    var x, y
    if (elem.style.left) {
      x = parseInt(elem.style.left)
    } else {
      x = 0
    }
    if (elem.style.top) {
      y = parseInt(elem.style.top)
    } else {
      y = 0
    }
    if (x == final_x && y == final_y) {
      return true
    }
    var dist = 10 //我们移动的距离
    if (x < fianl_x) {
      //如果横向在目标位置左侧，则+10向目标位置前进
      x = x + 10
    }
    if (x > final_x) {
      //如果横向在目标位置右侧，则-10向目标位置后退
      x = x - 10
    }
    if (y < fianl_y) {
      //同上
      y = y + 10
    }
    if (y > fianl_y) {
      y = y - 10
    }
  }, 25)
  elem.style.left = x + 'px'
  elem.style.top = y + 'px'
}
```

但是这串代码既不健壮也不简洁，还可以优化。

我们可以先简化一下代码：

```js
function animate(elem, final_x, final_y) {
  setInterval(function() {
    var x = parseInt(elem.style.left) || 0
    var y = parseInt(elem.style.top) || 0
    if (x === final_x && y === final_y) {
      return true
    }
    var dist = 10
    if (x < fianl_x) {
      //如果横向在目标位置左侧，则+10向目标位置前进
      x += 10
    }
    if (x > final_x) {
      //如果横向在目标位置右侧，则-10向目标位置后退
      x -= 10
    }
    if (y < fianl_y) {
      //同上
      y += 10
    }
    if (y > fianl_y) {
      y -= 10
    }
    elem.style.left = x + 'px'
    elem.style.top = y + 'px'
  }, 25)
}
```

问题 1. 移动距离每次都是固定的(10px)，是匀速移动。我们想逼真一点，当距离目标位置越近时速度变得越慢，产生渐变的效果；

问题 2. 定时器没有及时清除。

假如你先想通过调用 animate 函数将 elem 移动到目标位置 1，在其还未到达的时候，又改变主意让它移动到目标位置 2，再调用 animate 函数，传入了新的 final_x,final_y，此时就会造成两个定时器同时运作，产生混乱。

定时器 A 想让它到目标位置 1，定时器 B 想让它到目标位置 2,获取的元素当前位置却是同一个，就会造成 elem 摇摆不定，所以要以后一次的调用为准，清除前一次的定时器。对了，到达目标位置后也要清除，总不能把人家白白搁在那。

要养成好习惯：要用定时器，先清定时器。用完定时器，清除定时器。

首先解决问题 1。

```js
var distx = (final_x - x) / 10
var disty = (final_y - y) / 10
```

此时，我们的移动的距离不再是相同的 10px，而是根据目标位置与当前位置的差（距离差）来进行动态计算，当前位置距离目标位置越近我们移动的距离越小,每次移动距离差的十分之一，这样我们就不再是匀速运动了，距离差在不断减小，我们移动距离 dist 也在不断减小。
那些 if 也让人看着生厌，这时我们的 distx,disty 已经带有正负号，可以直接加在 x,y 上了。  
/10 要有取整处理，正负数的向上取整不同。代码如下：

```js
var distx = (final_x - x) / 10
distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx)
x = x + distx
var disty = (final_y - y) / 10
disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty)
y = y + disty
```

问题 1 结束。

其次解决问题 2。

我们可以知道，多个定时器都是作用在同一个元素上面，我们可以将每次进行的定时器的 ID 绑定在元素上，下一次执行新的定时器时将元素本身的定时器 clear（也就是上一个定时器）,再将此次定时器 ID 绑定即可。

最终代码：

```js
function animate(elem, final_x, final_y) {
  clearInterval(elem.timer) //要用定时器，先清定时器
  elem.timer = setInterval(function() {
    var x = parseInt(elem.style.left) || 0
    var y = parseInt(elem.style.top) || 0
    if (x === final_x && y === final_y) {
      clearInterval(elem.timer) //用完定时器，清除定时器
      return true
    }
    var distx = (final_x - x) / 10
    distx = distx > 0 ? Math.ceil(distx) : Math.floor(distx)
    x = x + distx
    var disty = (final_y - y) / 10
    disty = disty > 0 ? Math.ceil(disty) : Math.floor(disty)
    y = y + disty
    elem.style.left = x + 'px'
    elem.style.top = y + 'px'
  }, 25)
}
```

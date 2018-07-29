---
title: 函数节流与函数防抖
date: 2017-09-12
tags: [原生js]
categories: js
---

> 初稿：2017-07-15 <br>
> 更新: 2018-07-28

# 函数节流与函数防抖

函数防抖一部分我是学习于《JavaScript 高级程序设计》第三版，但是文中标题错误标为函数节流，实则为函数防抖。

网上大部分文章也是节流[throttle]防抖[debounce]分不清楚，其实这两个名称是极其形象的。

throttle-函数节流：一个水龙头在滴水，可能一次性会滴很多滴，但是我们只希望它每隔 500ms 滴一滴水，保持这个频率。即我们希望函数在以一个可以**接受**的频率重复调用。

debounce-函数防抖：将一个弹簧按下，继续加压，继续按下，只会在最后放手的一瞬反弹。即我们希望函数只会调用一次，即使在这之前反复调用它，最终也只会调用一次而已。

希望大家看完全文能够返回来看一看这段总结。

## 函数防抖

### 为什么会有函数防抖

先看一段代码：

```js
window.onresize = function() {
  var div = document.getElementById('mydiv')
  div.style.height = div.offsetWidth + 'px'
  console.log('resize')
}
```

很明显，监听浏览器窗口的 resize 事件，并基于该事件改变页面布局。首先，计算 offsetWidth 属性，如果该元素或者页面上其他元素有非常复杂的 CSS 样式，那么这个过程将会很复杂。其次，设置某个元素的高度需要对页面进行回流来令改动生效。如果页面有很多元素同时应用了相当数量的 CSS 的话，这又需要很多运算,可以通过'resize'输出的次数来观察函数调用的次数。

> 浏览器中某些计算和处理要比其他的昂贵很多。例如，DOM 操作比起非 DOM 交互需要更多的内存和 CPU 时间。连续尝试进行过多的 DOM 相关操作可能会导致浏览器挂起，有时候甚至会崩溃。

很明显，用户如果不断放大缩小浏览器窗口，那我们监听函数将会不停的被调用，倘若函数过“重”，即假设如上文描述的一般，那么对浏览器的压力将会非常之大，其高频率的更改可能会让浏览器崩溃。

### 什么是函数防抖

但是你会发现在某些页面的时候，你*手指按住鼠标左键不动*拖拉扩大或缩小浏览器时，页面内部并没有同步变化，当你*松开手指*时，页面才重新根据浏览器窗口变化。

此处便用到了函数防抖的思想，不论用户如何放大缩小，在他结束的的那一刻才进行我们的方法调用即可，不论怎么按弹簧，放手的时候才回弹。

debounce-函数防抖：将一个弹簧按下，继续加压，继续按下，只会在最后放手的一瞬反弹。即我们希望函数只会调用一次，即使在这之前反复调用它，最终也只会调用一次而已。

### 基本思想

某些代码不可以在没有间断的情况下连续重复执行。第一次调用函数，创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用该函数时，它会清除前一次的定时器并设置另一个。如果前一个定时器已经执行过了，这个操作(清除定时器)就没有任何意义。然而，如果前一个定时器尚未执行，其实就是将其替换为一个新的定时器。目的是只有在执行函数的请求停止了一段时间之后才执行。

### 基本模式

摘自高级程序设计

```js
var processor = {
  timeoutId: null,

  //实际进行处理的方法
  performProcessing: function() {
    //实际执行代码
  },

  //初始处理调用的方法
  process: function() {
    clearTimeout(this.timeoutId)
    var that = this
    this.timeoutId = setTimeout(function() {
      that.performProcessing()
    }, 100)
  }
}

// 尝试开始执行
processor.process()
```

但是大部分人一般对下面这种使用方式更为熟悉；

```js
function debounce(fn) {
  var timer
  var _self = fn
  return function() {
    clearTimeout(timer)
    var args = arguments // fn所需要的参数
    var _me = this // 当前的this
    timer = setTimeout(function() {
      _self.call(_me, args)
    }, 200)
  }
}
```

通过`debounce(fn)`代替`fn`;

debounce 函数接受要防抖的函数作为参数，返回值为一个匿名函数。

返回的匿名函数首先清除之前的定时器。定时器代码使用 call()来确保方法在适当的环境中执行。

### 解决问题

让我们回到最开始的代码：

```js
window.onresize = function() {
  var div = document.getElementById('mydiv')
  div.style.height = div.offsetWidth + 'px'
  console.log('resize')
}
```

现在看来这段代码弊端就很明显了，我们需要对这个函数防抖一下。让用户拉拉拖拖结束过后 200ms 再改变样式，节省浏览器资源。

```js
function debounce(fn) {
  var timer
  var _self = fn
  return function() {
    clearTimeout(timer)
    var args = arguments // fn所需要的参数
    var _me = this // 当前的this
    timer = setTimeout(function() {
      _self.call(_me, args)
    }, 200)
  }
}

function resizeDiv() {
  var div = document.getElementById('mydiv')
  div.style.height = div.offsetWidth + 'px'
  console.log('resize')
}

window.onresize = debounce(resizeDiv)
```

## 函数节流

### 为什么会有函数节流

基于上文的场景，我们完成了用户 resize 浏览器后 mydiv 的高度与宽度一致的这个需求。

这时候产品来了，说“你们这个有点奇怪呀，我放手之后他才变化，有点突兀，我在拖动的时候不能也让他变化一致吗？”

这就很尴尬了，好不容易想出来的防抖，就这样 pass 了？不，轮到节流登场了。

### 什么是函数节流

需求：优化用户体验，我们需要用户在 resize 浏览器窗口的过程中，height 与 width 也能保持一致，时刻触发函数肯定是不可以的，所以需要优化频率。

resize 过程中如果重复调用一个函数，让其以 500ms 的间隔执行，而非重复执行。

throttle-函数节流：一个水龙头在滴水，可能一次性会滴很多滴，但是我们只希望它每隔 500ms 滴一滴水，保持这个频率。即我们希望函数在以一个可以**接受**的频率重复调用。

### 基本模式

```js
function throttle(fn, interval) {
  var _self = fn
  var firstTime = true
  var timer

  return function() {
    var args = arguments
    var _me = this
    if (firstTime) {
      _self.call(_me, args)
    }

    if (timer) {
      return false
    }

    timer = setTimeout(function() {
      clearTimeout(timer)
      timer = null
      _self.call(_me, args)
    }, interval || 500)
  }
}
```

### 解决问题

```js
function throttle(fn, interval) {
  var _self = fn // 保存需要被延迟执行的函数引用
  var firstTime = true // 是否初次调用
  var timer // 定时器

  return function() {
    var args = arguments
    var _me = this
    if (firstTime) {
      // 如果是第一次调用不需要延迟执行
      _self.call(_me, args)
    }

    if (timer) {
      // 如果定时器还在，说明前一次延迟执行还没有完成
      return false
    }

    timer = setTimeout(function() {
      // 延迟一段时间执行
      clearTimeout(timer) // 清除定时器 避免下一次return false
      timer = null
      _self.call(_me, args)
    }, interval || 500)
  }
}

function resizeDiv() {
  var div = document.getElementById('mydiv')
  div.style.height = div.offsetWidth + 'px'
  console.log('resize')
}

window.onresize = throttle(resizeDiv)
```

现在我们即保证了用户体验也完成了性能保证。

---

参考书籍：《JavaScript 高级程序设计第三版》《JavaScript 设计模式与开发实践》

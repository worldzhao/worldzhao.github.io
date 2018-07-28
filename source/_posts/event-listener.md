---
title: 兼容性之事件注册
date: 2017-07-15
tags: [原生js]
categories: js
---

### 为什么要讨论事件注册的兼容性？

我原来写代码总是想着世界是进步的，一些落后的东西终将被淘汰，但是兼容性一直都是前端工程师在处理的问题，似乎用"世界进步"这种无所谓的语气偷懒也不太恰当了，而且此处讨论的事件注册不仅仅是兼容性问题，有时候又是需求需要的了。

### 事件注册有哪些方法？

- on+"event",例如:onclick/onmouseover/onmouseenter/...支持最广，笔者用的最多，倘若要在一个元素上添加多次同一事件，此时就显得无能为力了，以最后一次绑定的事件为准。

- addEventListener,W3C 标准方法，功能也最强大，支持添加多个事件

```js
//element.addEventListener(type,listener,useCapture);
obj.addEventListener('click', method1, false)
obj.addEventListener('click', method2, false)
obj.addEventListener('click', method3, false)
```

执行顺序为 method1->method2->method3，第三个参数是指以“冒泡”还是“捕获”的标准绑定事件，一般为 false(冒泡).

并且可以使用 removeEventListener() 方法移除由  addEventListener()方法添加的事件句柄。

**注意：**  如果要移除事件句柄，addEventListener() 的执行函数必须使用外部函数，如上实例所示 (method1/2/3)。
匿名函数，类似 "document.removeEventListener("_event_", function(){ *myScript* });" 该事件是无法移除的。

如果浏览器不支持 removeEventListener() 方法，你可以使用 detachEvent() 方法实现。

```js
var x = document.getElementById('myDIV')
if (x.removeEventListener) {
  // // 所有浏览器，除了 IE 8 及更早IE版本
  x.removeEventListener('mousemove', myFunction)
} else if (x.detachEvent) {
  // IE 8 及更早IE版本
  x.detachEvent('onmousemove', myFunction)
}
```

- attachEvent,IE 家的方法，火狐与其他家浏览器都不支持,attachEvent——兼容：IE7、IE8；不兼容 firefox、chrome、IE9、IE10、IE11、safari、opera.尽量不要用，支持绑定多个事件，与 addEventListener()执行顺序相反，即 method3->method2->method1

下面我们来进行兼容性处理：

1.  通过 if 判断

```js
if (document.addEventListener) {
  //功能最强大
  div.addEventListener('click', function() {
    alert('hello,world')
  })
} else if (document.attachEvent) {
  //非标准特性 尽量不要使用
  div.attachEvent('click', function() {
    alert('hello,world')
  })
} else {
  //支持最好
  div['onclick'] = function() {
    alert('hello,world')
  }
}
```

2.  封装成函数

```js
function registeEvent(elem, type, handler, useCapture) {
  if (document.addEventListener) {
    //功能最强大
    elem.addEventListener(type, handler, useCapture)
  } else if (document.attachEvent) {
    //非标准特性 尽量不要使用
    elem.attachEvent(type, handler)
  } else {
    //支持最好
    elem['on' + type] = handler
  }
}
```

到这里，我们每次注册事件时都通过 registeEvent 注册，很明显，每次注册都要判断浏览器的能力是否支持，每一次都要检测，这不是我们想要的。

3.  可以通过一个立即执行函数解决这个问题。

```js
var registeEvent = (function createEventRegister() {
  if (document.addEventListener) {
    return function(elem, type, handler, useCapture) {
      elem.addEventListener(type, handler, useCapture)
    }
  } else if (document.attachEvent) {
    return function(elem, type, handler) {
      elem.attachEvent(type, function() {
        handler.call(elem, window.event) //注:attachEvent内部this指向window而不是触发对象,使用call方法修改this
      })
    }
  } else {
    return function(elem, type, handler) {
      elem['on' + type] = handler
    }
  }
})()
```

此后只用通过 registeEvent 方法注册事件即可，且只在最开始的时候进行一次能力检测.

```js
registeEvent(div, 'click', function() {
  alert('hello,world')
})
```

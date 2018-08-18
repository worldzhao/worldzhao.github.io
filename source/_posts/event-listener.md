---
title: 实现通用的事件注册方法
date: 2017-07-15
tags: [兼容性]
categories: js
---

> 初稿：2017-07-15 <br>
> 更新: 2018-07-29

# 实现通用的事件注册方法

## 为什么要讨论事件注册的兼容性？

由于历史原因，不同的浏览器对事件注册的实现方式有些许差异。

## 事件注册有哪些方法？

### on+"event"

例如:onclick/onmouseover/onmouseenter/...支持最广，笔者用的最多，倘若要在一个元素上添加多次同一事件，此时就显得无能为力了，以最后一次绑定的事件为准。

### addEventListener

W3C 标准方法，功能也最强大，支持添加多个事件

```js
//element.addEventListener(type,listener,useCapture);
obj.addEventListener('click', method1, false)
obj.addEventListener('click', method2, false)
obj.addEventListener('click', method3, false)
```

执行顺序为 method1->method2->method3，第三个参数是指以“冒泡”还是“捕获”的标准绑定事件，一般为 false(冒泡).

并且可以使用 removeEventListener() 方法移除由  addEventListener()方法添加的事件句柄。

**注意：**  如果要移除事件句柄，addEventListener() 的执行函数必须使用外部函数，如上实例所示 (method1/2/3)。
匿名函数，类似 "window.removeEventListener("_event_", function(){ *myScript* });" 该事件是无法移除的。

如果浏览器不支持 removeEventListener() 方法，你可以使用 detachEvent() 方法实现。

```js
var x = window.getElementById('myDIV')
if (x.removeEventListener) {
  // // 所有浏览器，除了 IE 8 及更早IE版本
  x.removeEventListener('mousemove', myFunction)
} else if (x.detachEvent) {
  // IE 8 及更早IE版本
  x.detachEvent('onmousemove', myFunction)
}
```

### attachEvent

IE 家的方法，火狐与其他家浏览器都不支持,attachEvent——兼容：IE7、IE8；不兼容 firefox、chrome、IE9、IE10、IE11、safari、opera.尽量不要用，支持绑定多个事件，与 addEventListener()执行顺序相反，即 method3->method2->method1

## 进行兼容性处理

### 1.简单通过 if 判断

```js
if (window.addEventListener) {
  //功能最强大
  div.addEventListener('click', function() {
    alert('hello,world')
  })
} else if (window.attachEvent) {
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

### 2.封装成函数

```js
function registeEvent(elem, type, handler, useCapture) {
  if (window.addEventListener) {
    //功能最强大
    elem.addEventListener(type, handler, useCapture)
  } else if (window.attachEvent) {
    //非标准特性 尽量不要使用
    elem.attachEvent(type, handler)
  } else {
    //支持最好
    elem['on' + type] = handler
  }
}
```

到这里，我们每次注册事件时都通过 registeEvent 注册，很明显，每次注册都要判断浏览器的能力是否支持，每一次都要检测，这不是我们想要的。

### 3. 使用立即执行函数进行优化

```js
var registeEvent = (function createEventRegister() {
  if (window.addEventListener) {
    return function(elem, type, handler, useCapture) {
      elem.addEventListener(type, handler, useCapture)
    }
  } else if (window.attachEvent) {
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

但是此时依旧存在一个缺点，如果我们从头到尾没有绑定过事件，即使用 registeEvent 函数，那么立即执行函数白白执行了一次完全是多余的(或许有些吹毛求疵)，我们可以使用惰性载入函数来进行优化。

### 4. 惰性载入函数方案

此时 registerEvent 依然被声明为一个普通函数，在函数里依然有一些分支判断。但是在第一次进入条件分支之后，在函数内部会重写这个函数，重写之后的函数就是我们期望的 registerEvent 函数，在下一次进入 registerEvent registerEvent 函数里不再存在条件分支语句:

```js
var registeEvent = function createEventRegister() {
  if (window.addEventListener) {
    registeEvent = function(elem, type, handler, useCapture) {
      elem.addEventListener(type, handler, useCapture)
    }
  } else if (window.attachEvent) {
    registeEvent = function(elem, type, handler) {
      elem.attachEvent(type, function() {
        handler.call(elem, window.event) //注:attachEvent内部this指向window而不是触发对象,使用call方法修改this
      })
    }
  } else {
    registeEvent = function(elem, type, handler) {
      elem['on' + type] = handler
    }
  }
}
```

---

附上一些兼容性解决方案 EventUtil：

```js
var EventUtil = {
  addEvent: function(element, type, handler) {
    // 添加绑定
    if (element.addEventListener) {
      // 使用DOM2级方法添加事件
      element.addEventListener(type, handler, false)
    } else if (element.attachEvent) {
      // 使用IE方法添加事件
      element.attachEvent('on' + type, handler)
    } else {
      // 使用DOM0级方法添加事件
      element['on' + type] = handler
    }
  },
  // 移除事件
  removeEvent: function(element, type, handler) {
    if (element.removeEventListener) {
      element.removeEventListener(type, handler, false)
    } else if (element.datachEvent) {
      element.detachEvent('on' + type, handler)
    } else {
      element['on' + type] = null
    }
  },
  getEvent: function(event) {
    // 返回事件对象引用
    return event ? event : window.event
  },
  // 获取mouseover和mouseout相关元素
  getRelatedTarget: function(event) {
    if (event.relatedTarget) {
      return event.relatedTarget
    } else if (event.toElement) {
      // 兼容IE8-
      return event.toElement
    } else if (event.formElement) {
      return event.formElement
    } else {
      return null
    }
  },
  getTarget: function(event) {
    //返回事件源目标
    return event.target || event.srcElement
  },
  preventDefault: function(event) {
    //取消默认事件
    if (event.preventDefault) {
      event.preventDefault()
    } else {
      event.returnValue = false
    }
  },
  stoppropagation: function(event) {
    //阻止事件流
    if (event.stoppropagation) {
      event.stoppropagation()
    } else {
      event.canceBubble = false
    }
  },
  // 获取mousedown或mouseup按下或释放的按钮是鼠标中的哪一个
  getButton: function(event) {
    if (document.implementation.hasFeature('MouseEvents', '2.0')) {
      return event.button
    } else {
      //将IE模型下的button属性映射为DOM模型下的button属性
      switch (event.button) {
        case 0:
        case 1:
        case 3:
        case 5:
        case 7:
          //按下的是鼠标主按钮（一般是左键）
          return 0
        case 2:
        case 6:
          //按下的是中间的鼠标按钮
          return 2
        case 4:
          //鼠标次按钮（一般是右键）
          return 1
      }
    }
  },
  //获取表示鼠标滚轮滚动方向的数值
  getWheelDelta: function(event) {
    if (event.wheelDelta) {
      return event.wheelDelta
    } else {
      return -event.detail * 40
    }
  },
  // 以跨浏览器取得相同的字符编码，需在keypress事件中使用
  getCharCode: function(event) {
    if (typeof event.charCode == 'number') {
      return event.charCode
    } else {
      return event.keyCode
    }
  }
}
```

参考书籍：《JavaScript 高级程序设计第三版》《JavaScript 设计模式与开发实践》
参考文章：[手把手教你用原生 JavaScript 造轮子（1）——分页器](https://juejin.im/post/5b592635e51d4533d2043e15?utm_source=gold_browser_extension)

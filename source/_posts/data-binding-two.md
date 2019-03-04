---
title: 设计模式-发布订阅模式
date: 2017-07-24
tags: [发布订阅模式]
categories: 设计模式
---

### 前言

昨天学习了通过 Object.defineProperty()对象属性绑定 getter 与 setter，从而检测对象属性的变化，但是我们并不仅仅满足于检测到变化，我们还要做出行为(回调)，即实现 vue 中的 watch 这个 api,由此，引出了发布-订阅模式，每一次属性变化发生时(发布)，都会执行对应的函数(订阅)，这两天也看了不少的相关文章，记录一二。

> 以下概念部分转自[博客园:龙恩 0707 的博客](http://www.cnblogs.com/tugenhua0707/p/4687947.html)，本人略微有所改动，代码部分参考了[vue 源码学习之发布订阅实现\$watch](https://segmentfault.com/a/1190000008808297),但是我感觉原博的代码好像有一点问题，自己改了些许。

## 发布订阅模式

### 定义

它定义了对象间的一种一对多的关系，让多个观察者对象同时监听某一个主题对象，当一个对象发生改变时，所有依赖于它的对象都将得到通知。

### 现实中的发布订阅模式

比如小红最近在淘宝网上看上一双鞋子，但是呢 联系到卖家后，才发现这双鞋卖光了，但是小红对这双鞋又非常喜欢，所以呢联系卖家，问卖家什么时候有货，卖家告诉她，要等一个星期后才有货，卖家告诉小红，要是你喜欢的话，你可以收藏我们的店铺，等有货的时候再通知你，所以小红收藏了此店铺，但与此同时，小明，小花等也喜欢这双鞋，也收藏了该店铺；等来货的时候就依次会通知他们；

在上面的故事中，可以看出是一个典型的发布订阅模式，卖家是属于发布者，小红，小明等属于订阅者，订阅该店铺，卖家作为发布者，当鞋子到了的时候，会依次通知小明，小红等，依次使用旺旺等工具给他们发布消息。

### 发布订阅模式的优点

1.  支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象。
    比如上面的列子，小明，小红不需要天天逛淘宝网看鞋子到了没有，在合适的时间点，发布者(卖家)来货了的时候，会通知该订阅者(小红，小明等人)。

2.  发布者与订阅者耦合性降低，发布者只管发布一条消息出去，它不关心这条消息如何被订阅者使用，同时，订阅者只监听发布者的事件名，只要发布者的事件名不变，它不管发布者如何改变；同理卖家（发布者）它只需要将鞋子来货的这件事告诉订阅者(买家)，他不管买家到底买还是不买，还是买其他卖家的。只要鞋子到货了就通知订阅者即可。

对于第一点，我们日常工作中也经常使用到，比如我们的 ajax 请求，请求有成功(success)和失败(error)的回调函数，我们可以订阅 ajax 的 success 和 error 事件。我们并不关心对象在异步运行的状态，我们只关心 success 的时候或者 error 的时候我们要做点我们自己的事情就可以了~

### 发布订阅模式的缺点：

创建订阅者需要消耗一定的时间和内存。
虽然可以弱化对象之间的联系，如果过度使用的话，反而使代码不好理解及代码不好维护等等。

### 如何实现发布订阅模式？

1.  首先要想好谁是发布者(比如上面的卖家，比如 setter)。
2.  然后给发布者添加一个缓存列表(eventBus)，用于存放回调函数来通知订阅者(比如上面的买家收藏了卖家的店铺，卖家通过收藏了该店铺的一个列表名单)。
3.  最后就是发布消息，发布者遍历这个缓存列表，依次触发里面存放的订阅者回调函数。

目标如下：

```js
let app1 = new Observer({
  name: 'youngwind',
  age: 25
})

// 你需要实现 $watch 这个 API
app1.$watch('age', function(age) {
  console.log(`我的年纪变了，现在已经是：${age}岁了`)
})

app1.data.age = 100 // 输出：'我的年纪变了，现在已经是100岁了'
```

先把昨天的代码摆出来：

```js
class Observer {
  //构造函数
  constructor(data) {
    this.data = data
    this.walk(data)
  }
  //类的方法(原型方法)
  walk(obj) {
    let val
    for (let key in obj) {
      //hasOwnProperty过滤属性，判断属性是不是对象自身属性而非对象原型属性
      if (obj.hasOwnProperty(key)) {
        val = obj[key]
        //如果属性还是一个对象，进行递归
        if (Object.prototype.toString.call(val) === '[object Object]') {
          new Observer(val)
        }
        //为每一个属性添加getter以及setter
        this.convert(key, val)
      }
    }
  }
  convert(key, val) {
    Object.defineProperty(this.data, key, {
      enumerable: true,
      configurable: true,
      get() {
        console.log('你访问了' + key)
        return val
      },
      set(newVal) {
        console.log('你设置了' + key)
        console.log('新的' + key + '=' + newVal)
        if (newVal === val) return
        val = newVal
      }
    })
  }
}
let obj = {
  user: {
    name: 'zac',
    age: '20'
  },
  address: {
    city: 'wh'
  }
}
new Observer(obj)
console.log(obj) // 现在可以去控制台做有趣的事情啦
```

- 思路：

1.  发布订阅模式实现一个观察者；
2.  将观察者挂在 app 上；

先加上\$watch 这个 api

```js
class Observer{
  ...
  $watch(key,callback){

  }
}
```

## 实现一个订阅器

Event.js

```js
export default class Events {
  constructor() {
    this.events = {} //键:检测的属性名 值:当对应的属性发生变化时应作出的相应操作，即相应的函数，可能有多个（不止一个订阅）,所以类型为数组。
  }
  // 订阅操作 $watch内部进行
  on(prop, callback) {
    if (!this.events[prop]) {
      this.events[prop] = [] //回调函数数组
    }
    this.events[prop].push(callback)
    return this
  }
  // 解除订阅
  remove(prop) {
    for (var key in this.events) {
      if (this.events.hasOwnProperty(key) && key === prop) {
        delete this.events[prop]
      }
    }
  }
  // 通知遍历并执行对应属性的回调函数数组
  emit(prop, val, newVal) {
    if (!this.events[prop]) {
      //判断是否订阅过如果没有订阅该属性，返回
      return this
    }
    var args = Array.prototype.slice.call(arguments, 1) //取出参数 此处为val newVal
    for (let i = 0; i < this.events[prop].length; i++) {
      //执行属性对应的回调函数（数组）
      this.events[prop][i].apply(this, args) //将可能用到的值传入
    }
    return this
  }
}
```

以下为全部代码：

```js
import Events from './Events'
class Observer {
  constructor(data) {
    this.data = data
    this.walk(this.data)
  }
  walk(data) {
    let val
    for (let key in data) {
      //hasOwnProperty过滤属性，判断属性是不是对象自身属性而非对象原型属性
      if (data.hasOwnProperty(key)) {
        val = data[key]
        //如果属性还是一个对象，进行递归
        if (Object.prototype.toString.call(val) === '[object Object]') {
          new Observer(val)
        }
        //为每一个属性添加getter以及setter
        this.convert(key, val)
      }
    }
  }
  convert(key, val) {
    let that = this
    Object.defineProperty(this.data, key, {
      enumerable: true,
      configurable: true,
      get() {
        console.log('你访问了' + key)
        return val
      },
      set(newVal) {
        console.log('你设置了' + key)
        console.log('新的' + key + '=' + newVal)
        if (newVal === val) return
        if (Object.prototype.toString.call(newVal) === '[object Object]') {
          new Observer(newVal) //如果新设置的值是对象，则需要递归
        }
        val = newVal
        that.eventsBus.emit(key, val, newVal) //发布
      }
    })
  }
  $watch(key, callback) {
    this.eventsBus.on(key, callback) //订阅
  }
}
Observer.prototype.eventsBus = new Events() //所有的Observer实例共享这一个eventsBus，这一步很重要，倘若每一个对象或子对象都有自己的eventsBus，则可能在父对象上eventsBus.on，却在子对象eventsBus.emit，回调函数此时在父对象的eventsBus属性内，无法触发。表达可能有点混乱，见谅

let data = {
  user: {
    name: 'zac',
    age: '20'
  },
  address: {
    city: 'wh'
  }
}

let obj = new Observer(data)

obj.$watch('age', function(val, newVal) {
  console.log(`我的年纪变了，现在已经是：${newVal}岁了`)
})

vm.data.user.age = 100 // 输出:你设置了 age，新的值为100 我的年纪变了，现在已经是：100岁了
```

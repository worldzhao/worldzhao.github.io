---
title: this、箭头函数与React事件绑定（一）
date: 2017-10-13
tags: [this]
categories: js
---

# 函数调用模式

函数有 4 种调用模式:

1. 方法调用模式(对象)
2. 函数调用模式(window 对象)
3. 构造器调用模式(new)
4. 上下文模式

了解函数调用模式的不同,对于我们理解函数内部 this 指向有着重要的作用

## 方法调用模式与函数调用模式

demo1:

```js
var age = 38
var obj = {
  age: 18,
  getAge: function() {
    return this.age
  }
}
console.log(obj.getAge()) //18  方法调用模式

var getAge = obj.getAge
console.log(getAge()) //38   函数调用模式
```

demo2:

```js
var age = 38
var obj = {
  age: 18,
  getAge: function() {
    //方法调用 this指向调用对象
    console(this.age)
    function foo() {
      console.log(this.age)
    }
    foo() //函数调用,this指向window对象(非严格模式)
  }
}

obj.getAge() // 18  38
```

demo3:

```js
var length = 10
function fn() {
  console.log(this.length)
}

var obj = {
  length: 5,
  method: function(fn) {
    fn() //函数调用 10
    arguments[0]() // 方法调用 arguments[0] => arguments.fn  this 指向 arguments length为2
  }
}
obj.method(fn, 123) //10 2
```

相当于 arguments 调用了 fn 返回 arguments 的长度

> arguments 是一个包含很多属性的对象,键名为索引,键值为传进来的参数值

获取对象的属性值有两种方法:点方法与中括号法 obj.name === obj['name']

arguments[0] === arguments.0(并不能这样写)

调用 arguments.0\(\)==arguments\[0\]\(\) 实际上是方法调用

理解不了看下面:

```js
var obj = { abc: fn }
obj.abc() //方法调用
obj['abc']() //方法调用
```

## 构造器调用模式

demo4:

```js
function Person(name, age) {
  this.name = name
  this.age = age
}
var p = new Person('zzw', '18')
```

new 关键字就做了一点微小的工作

```js
var obj={};
obj.\_\_proto\_\_=Person.prototype;
Person.call(obj);//通过call方法强行将构造函数内部的this指向了obj
```

## 上下文模式

首先来看代码:

```js
var p1 = {
  name: '赵志文',
  age: '18'
}

var name = '周娇娇'

function getName() {
  console.log(this.name, this)
}

getName() //周娇娇 window //函数调用,this指向window(非严格模式)

getName.call(p1) //赵志文 Object //改变了getName内部的this指向,指向p1

getName.apply(p1) //同上
```

相信各位读者通过上例能够很轻易的知道 call 与 apply 的作用:改变调用函数内部的 this 指向.

**使用 call 与 apply,可以修改函数调用上下文,也就是 this 的值,这两个方法都是定义在 Function.prototype 中,所有函数都可以调用**

### Function.prototype.apply/call

MDN 文档中写道:"apply()方法在指定 this 值和参数(参数以数组或类数组形式存在)的情况下调用某个函数."

> apply()方法与 call()方法仅有的区别:call()方法接收的是一个参数列表,而 apply()方法接收的是一个包含多个参数的数组(或类数组对象);

语法:

```js
fun.apply(thisArg [,argsArray]);
fun.call(thisArg,para1,para2,...);
```

例子

```js
var name = 'zzw'

function test(a, b) {
  console.log(this.name + '吃了' + (a + b) + '个西瓜')
}

test(1, 2) // 函数调用模式 非严格模式下指向window zzw吃了3个西瓜

var obj = {
  name: 'zjj'
}

var obj1 = {
  name: 'ruizhi'
}

test.call(obj, 4, 5) // 改变this指向为obj同时传参  zjj吃了9个西瓜
test.apply(obj1, [100, 99]) // 改变this指向为obj同时传参 ruizhi吃了199个西瓜
```

### 举例

- 求一个数组中的最大值

> js 中 Math 对象有个方法 max,求出参数中的最大值
> 但是往往给定的是数组 ,怎么办捏
> 使用 apply,这里就不改变 this 了,单纯数组转参数

知识点:apply 与 call 第一个参数为 null 时,单纯调用函数,着眼于参数类型(例如给定数组,通过 apply 数组转参数列表)

```js
var arr = [1, 20, 157, 76, 84]
var max = Math.max.apply(null, arr)
console.log(max)
```

- 将传入函数的参数打印出来,用'-'连接

法一:自己的傻瓜拼接字符串法

```js
function demo4() {
  var res = ''
  for (var j = 0; j < arguments.length; j++) {
    if (j < arguments.length - 1) {
      res += arguments[j] + '-'
    } else {
      res += arguments[j]
    }
  }
  console.log(res)
}
demo4(1, 2, 3, 'abc', 'zzw') //1-2-3-abc-zzw
```

法二:可以重新创建一个数组容纳参数,调用 join 方法

```js
function demo4() {
  var arr = []
  for (var i = arguments.length - 1; i >= 0; i--) {
    arr.unshift(arguments[i])
  }
  arr = arr.join('-')
  console.log(arr)
}
demo4(1, 2, 3, 'abc', 'zzw') //1-2-3-abc-zzw
```

法三:直接借用 join 方法,改变 join 的内部 this 指向

```js
function demo4() {
  //return arguments.join("-");//报错 伪数组并非数组 不可以使用join
  var str = Array.prototype.join.call(arguments, '-') //改变join内部的this
  console.log(str)
  //或写成 var str = [].join.call(arguments,"-");
}
demo4(1, 2, 3, 'abc', 'zzw') //1-2-3-abc-zzw
```

### this 与构造函数

在学习构造函数的过程中肯定会思考构造函数中的 this 到底是什么呢？

```js
function Person(name, job) {
  //默认隐含的操作,将new创建的对象赋值给this
  this.name = name
  this.job = job
  this.sayHello = function() {
    console.log('hello')
  }
}

var winter = new Person('winter', 'coding')
```

构造函数的执行过程:

- 使用 new 关键字创建对象;
- 调用构造函数,将新创建出来的对象赋值给对象函数内部的 this;//如何做到?
- 在构造函数内部使用 this 为新创建的对象新增成员;
- 默认返回新创建的对象,普通函数如果不写返回语句,会返回 undefined.

> new 关键字就做了一点微小的工作

```js
function new(F){
    return function () {
        var args = arguments;
        var obj = {}
        obj.__proto__ = F.prototype
        F.call(obj,args)
        return obj
    }
}
```

此处也是 call 与 apply 的一个典型用法.

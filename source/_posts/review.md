---
title: 复习及面向对象初步
date: 2017-07-04
tags: [原生js]
categories: js
---

## JavaSript 包含三大部分

* ECMAScript:规定 js 的语法规法

* DOM: Document Object Model 文档对象模型,给我们提供了一套完整的操作页面 api

* BOM: Browser Object Model 浏览器对象模型

## 数据类型

* 基本数据类型: string number boolean undefined

* 复杂数据类型(引用类型): Date Array Object RegExp String Number Boolean Function Math Null 等...

```JS
null == undefined // true
null === undefined // false
```

## in 关键字

* 用于 for in 循环

* 判断属性是否存在于对象中语法： 属性名 in 对象属性名要用引号

```js
var isExist = 'name' in obj
```

* 数组用法 判断索引是否存在,而非值 "index in arr"
  console.log(0 in arr);
  0 是 index,也可用 console.log("0" in arr);
  此处会进行隐式转换,无问题

## 值传递与引用传递

首先,形参和实参并不是同一个值

其次,形参与实参之间存在值传递和引用传递的过程,是一个复制(赋值)的过程.

值传递:形参单纯的将实参的值复制一遍,形参修改不影响到外部的实参值.

引用传递:形参获得实参存储的地址指向,在函数内部通过形参修改对象,则会改变外部对象.

注:倘若形参重新存储地址指向(创建对象),那么形参与实参则分别指向不同对象,相互操作不受影响

## 对象的动态特性

是指在对象创建出来之后,可以为对象动态添加新属性和新方法.

* 使用点语法,例如 obj.name,可以获取对象的属性值,也可以修改对象的属性值,还可以为对象新增属性并赋值. 例如 obj.name="xxx",进行赋值的时候,如果存在该属性则进行修改值.  
  如果对象不存在该属性,是给该对象添加属性或方法.

* 可以通过对象名["属性名]访问对象的属性值,注意这里的属性名是字符串,如果这里使用的不是字符串,那么会隐式地转变成字符串.
  例如 obj["name"],同时也可以也可以修改对象的属性值,还可以为对象新增属性并赋值.

此处给 obj 对象添加了一个属性,这个属性也是一个对象:

```js
var obj = {
  name: 'zzw',
  age: 18
}
obj['car'] = {
  brand: 'benz',
  price: 1000000
}
```

注:对象是不是键值对的集合？json 的属性名必须要加双引号,对象可以不用加.

总结:新增属性、方法的方式有： 1.点语法 2.通过[]的形式,必须使用字符串

## delete 关键字

delete 关键字可以用来删除对象的属性,以及不是用 var 声明的变量

执行完 delete 关键字会有返回值,删除成功返回 true 删除失败返回 false

特殊之处:删除不存在的属性返回 true

```js
var result = delete obj.gender // true
```

如果删除的属性存在于原型当中,那么返回值为 true 但是并未删除.

## 异常捕获

```js
try {
  可能出现异常的代码
} catch (e) {
  捕获异常后的执行代码
} finally {
  最终执行的代码
}
```

## 案例:面向对象编程举例 01-03

### 需求

给页面中的 div 与 p 标签加上 border，设置页面中 div 和 p 的边框为 1px solid red

#### 面向过程解决方式:

```js
var divs = document.querySelectorAll('div')
var ps = document.querySelectorAll('p')

for (var i = 0; i < divs.length; i++) {
  divs[i].style.border = '1px solid red'
}

for (var i = 0; i < ps.length; i++) {
  ps[i].style.border = '1px solid red'
}
//编写代码的原则: DRY don't repeat yourself
```

#### 函数封装解决方式

* 使用函数将代码封装,使得复用性更高
* 使用函数封装带来的问题: 1.全局变量污染 ;2.代码结构不够清晰,维护困难

```js
var divs = getElements('div')
var ps = getElements('p')

setStyle(divs)
setStyle(ps)

function getElements(tagName) {
  var elems = document.querySelectorAll(tagName)
  return elems
}

function setStyle(elems) {
  for (var i = 0; i < elems.length; i++) {
    elems[i].style.border = '1px solid red'
  }
}
```

#### 面向对象解决方式

使用对象封装后的优势

* 暴露在全局中的只有一个对象名,不会造成全局变量污染
* 是用对象将代码进行功能模块化的划分
* 有利于日后的维护

```js
var zQuery = {
  //使用zQuery对象属性根据功能对对象进行模块化 获取元素的zQuery属性对象
  getEle: {
    //1.获取元素的方法们
    tag: function (tagName) {
      return document.querySelectorAll(tagName);
    },
    id: function (id) {
      return document.getElementById(id);
    }
  },

  // 修改css样式的zQuery属性对象
  setCss: {
    //修改css样式的方法们
    setStyle: function (elems) {
      for (var i = 0; i < elems.length; i++) {
        elems[i].style.border = "1px solid red";
      }
    }
    css: function (option) {
      //...
    }
    addClass: function (className) {
      //...
    }
  }

};
var divs = zQuery.tag('div');
var ps = zQuery.tag('p');
zQuery.setStyle(divs);
zQuery.setStyle(ps);
```

* 面向对象三大特征:封装 继承 多态

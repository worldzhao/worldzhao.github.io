---
title: 最近遇到的一些js编程题
date: 2017-09-27
tags: [练习题]
categories: js
---

## 数组扁平化

例：
输入：[[1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14]]]], 10]
输出：[ 1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 9, 11, 12, 12, 13, 14, 10 ]

### 方法一

```js
var newArr = []
function flatten(arr) {
  for (var i = 0; i < arr.length; i++) {
    if (Object.prototype.toString.call(arr[i]) === '[object Array]') {
      flatten(arr[i])
    } else {
      newArr.push(arr[i])
    }
  }
}
```

### 方法二

```js
var flatten = function(array) {
  return array.reduce(function(previous, val) {
    if (Object.prototype.toString.call(val) !== '[object Array]') {
      previous.push(val)
      return previous
    }
    Array.prototype.push.apply(previous, flatten(val))
    return previous
  }, [])
}
```

### 方法三

```js
function flatten(arr) {
  while (arr.some(item => Array.isArray(item))) {
    arr = [].concat(...arr)
  }
  return arr
}
```

## 实现 map

```js
Array.prototype.myMap = function(callback) {
  var arr = []
  for (var i = 0; i < this.length; i++) {
    var item = callback(this[i], i)
    arr.push(item)
  }
  return arr
}
```

## 实现 bind

```js
Function.prototype.myBind = function(context) {
  var self = this
  var args = [].slice.call(arguments, 1)
  return function() {
    return self.apply(context, args.concat(arguments))
  }
}
```

## 实现一个简单的字符串模版引擎

效果如下：

```js
var str = 'hello, i am <%=user%>, from <%=location%>'
var compiled = template(str)
compiled({ user: 'zzw', location: 'ez' }) // 'hello, i am zzw, from ez'
```

代码：

```js
function template(temp) {
  var temp = temp
  return function(obj) {
    var reg = /<%=(\w+)%>/
    var result
    console.log(temp)
    while ((result = reg.exec(temp))) {
      var key = result[1]
      var value = obj[key]
      temp = temp.replace(result[0], value)
    }
    return temp
  }
}
```

## 实现深克隆

```js
function deepClone(obj) {
  var result,
    oClass = isClass(obj)
  //确定result的类型
  if (oClass === 'Object') {
    result = {}
  } else if (oClass === 'Array') {
    result = []
  } else {
    return obj
  }
  for (key in obj) {
    var copy = obj[key]
    if (isClass(copy) == 'Object' || isClass(copy) == 'Array') {
      result[key] = arguments.callee(copy) //递归调用
    } else {
      //如果为基本数据类型
      result[key] = obj[key]
    }
  }
  return result
}
//返回传递给他的任意对象的类
function isClass(o) {
  if (o === null) return 'Null'
  if (o === undefined) return 'Undefined'
  return Object.prototype.toString.call(o).slice(8, -1)
}
```

## 深克隆变形题目：将 json 字符串所有键名第一个首字母转为大写(考虑嵌套对象)

例：输入

```js
{"hyKey":"myValue","q23":"123","arr":[1,2,3],"obj":{"name":"zzw"},"null":null}
```

输出:

```js
{"HyKey":"myValue","Q23":"123","Arr":[1,2,3],"Obj":{"Name":"zzw"},"Null":null}
```

```js
function toUpperCase1(obj) {
  var result,
    oClass = isClass(obj)
  if (oClass === 'Object') {
    result = {}
  } else if (oClass === 'Array') {
    result = []
  } else {
    return obj
  }
  for (var key in obj) {
    var copy = obj[key]
    if (isClass(copy) == 'Object' || isClass(copy) == 'Array') {
      result[key.slice(0, 1).toUpperCase() + key.slice(1)] = arguments.callee(
        copy
      )
    } else {
      result[key.slice(0, 1).toUpperCase() + key.slice(1)] = obj[key]
    }
  }
  return result
}
```

## 事件委托

实现：给定一个 ul 列表，里面有若干个 li 标签，li 里面也嵌套了若干标签，要求点击标签，弹出当前 li 在 ul 中的位置.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <ul id="ul">
        <li><a href="#"><span>li-a-span</span></a></li>
        <a href="#">a</a>
        <li><a href="#"><span>li-a-span</span></a></li>
        <li><a href="#"><span>li-a-span</span></a></li>
        <li><a href="#"><span>li-a-span</span></a></li>
        <li><a href="#"><span>li-a-span</span></a></li>
    </ul>
</html>
```

```js
var ul = document.getElementById('ul')
ul.addEventListener('click', function(e) {
  // 事件委托
  var e = e.target
  while (e.nodeName != 'LI') {
    e = e.parentNode
  }
  var childrenArr = [].slice.apply(ul.children) // 伪数组转为数组 为了使用filter方法
  var liArr = childrenArr.filter(function(child) {
    if (child.nodeName == 'LI') {
      return child
    }
  })
  console.log(liArr.indexOf(e))
})
```

## 输出字符串中所有的叠词

输入： '晴川历历汉阳树，芳草萋萋鹦鹉洲'
输出： [ '历历', '萋萋' ]

```js
function rw(str) {
  var result = []
  for (var i = 1; i < str.length; i++) {
    if (str[i] == str[i - 1]) {
      result.push(str.slice(i - 1, i + 1))
    }
  }
  return result
}
```

## 实现一个 add 函数

执行如下：

```js
function add() {
  var sum = 0
  var arr = [].slice.call(arguments)
  for (var i = 0; i < arr.length; i++) {
    sum += arr[i]
  }
  var temp = function() {
    var arr = [].slice.call(arguments)
    for (var i = 0; i < arr.length; i++) {
      sum += arr[i]
    }
    return temp
  }

  temp.toString = function() {
    return sum
  }
  return temp
}
```

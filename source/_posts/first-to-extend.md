---
title: 简单继承
date: 2017-07-04
tags: [原生js]
categories: js
---
## 继承

### 混入式继承
最简单的继承,使用for in 循环为对象动态添加成员,代码如下:

```js
var obj1 = {
  name: "zzw",
  age: "18"
};

console.log(obj1);

var obj2 = {};

for (var k in obj1) {
  obj2[k] = obj1[k];
}

console.log(obj2);

```

### 原型式继承：Object.create
ES5为我们提供了一个方法Object.create(obj),返回值为一个继承于obj对象的对象

```js
var subObj = Object.create(superObj); // subObj继承了superObj的成员
```

内部实现：
```js
function create(superObj) {
  var F = function() {};
  F.prototype = superObj;
  return new F();
}

```
    

由于此方法是ES5提供的,所以存在兼容性问题.
如何解决兼容性问题呢?

检测浏览器能力.
```js
if (Object.create) {
  //若浏览器支持
  var subObj = Object.create(superObj);
} else {
  //若浏览器不支持,自己添加这个方法
  Object.create = function() {
    var F = function() {};
    F.prototype = superObj;
    return new F();
  };
}
```
        

封装成函数:
```js
function create(superObj) {
  if (Object.create) {
    return Object.create(superObj);
  } else {
    Object.create = function() {
      var F = function() {};
      F.prototype = superObj;
      return new F();
    };
  }
}
```
        


### 通过原型继承(此处不讨论原型链，单纯继承原型对象的成员)
* 给原型对象中添加成员(对象的动态特性,非严格意义上的继承)
```js
Person.prototype.sayHello = function() {
  console.log("hello");
};

var p = new Person("冯巩", "50");

p.sayHello(); //这里的p对象就继承了原型对象

```

* 直接替换原型对象,原型对象原本的属性无法保存
```js
var parent = {
  saygoodbye: function() {
    console.log("goodbye");
  }
};

Person.prototype = parent;

var p1 = new Person("zzw", "20");

p1.saygoodbye(); //p对象就继承了原型对象,也就是我们的parent对象

```   

* 用混入的方式为原型对象添加成员,保存了原型对象原有的属性(sayhi方法)
```js
Person.prototype.sayhi = function() {
  console.log("hi");
};

var parent = {
  saygoodbye: function() {
    console.log("goodbye");
  }
};

for (var k in parent) {
  Person.prototype[k] = parent[k];
}

var p2 = new Person("zzw", "18");

p2.saygoodbye();
p2.sayhi();

```

---
title: 从构造函数到原型
date: 2017-07-04
tags: [js基础]
categories: js
---

![constructor&prototype&instance.png](http://upload-images.jianshu.io/upload_images/4869616-7c3eaf862ec505e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 传统构造函数模式存在的问题

如果在构造函数中定义某个函数，那么每次创建新的对象,都会创建一遍该函数

```js
function Student(name) {
  this.name = name
  this.skill = function() {
    console.log('goodgoodstudy')
  }
}
var zzw = new Student('zzw')
var zjj = new student('zjj')
```

函数内部代码完全相同,而内存中存在了多个相同内容的函数,仅仅是地址不同,造成了资源浪费.

为了解决这个问题,我们要让所有对象共用一个方法,在构造函数外部定义好该函数.

将函数赋值给构造函数内部的方法,这样内存内只存在一个方法,对象内部的方法名全都指向该方法.

```js
function studyMethod() {
  console.log(this.name + 'goodgoodstudy')
}
function Student(name) {
  this.name = name
  this.skill = studyMethod
}
var zzw = new Student('zzw')
var zjj = new Student('zjj')
zzw.skill()
```

问题又来了:如果对象公用的方法很多,那么就在构造函数外部定义好很多个函数,全局变量增多,造成污染,代码结构混乱,不易维护.

由此引入原型模式.

## 原型模式

1.  原型是什么:在构造函数创建出来的时候,浏览器会默认为构造函数创建并关联一个特殊的**对象**,即原型.原型默认是一个空的对象,原型对象是构造函数(对象)的属性.
2.  原型可以用来做什么:原型对象的所有属性与方法都是被构造函数的实例对象所公用的.通过对象的动态特性为原型对象添加属性与方法.
3.  原型对象创建出来的时候会有个默认的属性 constructor 指向对应构造函数.
4.  原型对象创建出来的时候会有个默认的属性\_\_proto\_\_指向自身.
    对象如何访问原型?p.\_\_proto\_\_

- 原型对象添加成员的形式：

1.  利用对象的动态特性添加成员(需要添加的成员较少时)
2.  直接重写原型对象(需要添加的成员较多时)

```js
function Person(name, sex, age) {
  this.name = name
  this.age = age
  this.sex = sex
}

Person.prototype.sayHello = function() {
  console.log('hello')
} //为原型添加成员方法

var p1 = new Person('zzw', 'male', '20')

Person.prototype = {
  sayGoodBye: function() {
    console.log('byebye')
  }
} //替换原型

var p2 = new Person('zjj', 'female', '19')

p1.sayHello() //hello

p2.sayHello() //undefined
```

通过以上代码可以得知:

实例对象可以访问的原型对象的成员以**实例对象创建时**构造函数所关联的原型对象为主,倘若原型对象被**重写**,则重写之前实例化的实例对象依旧访问的是旧原型的成员(为原型新增成员不算重写).

- 使用原型模式的注意事项:

  1.当访问对象内部方法或属性时,首先在对象内部搜索,如果搜索不到则前往原型对象中查找;

  2.使用点语法进行赋值的时候,如果对象中不存在该属性(值类型而非引用类型),则给该对象新增该属性,而不会去修改原型中的成员.

### 原型模式存在的问题

1.  如果在原型中的属性是引用类型的属性,那么所有的实例对象共享该属性,并且一个实例对象修改了引用类型属性中的成员,其他对象也都会受影响.并不会新增该属性!!

2.  所以一般情况下,不会将属性放在原型对象中,原型对象中只会存放共有的方法,将实例对象各自的属性写在构造函数中,称为构造函数与原型组合模式.

## 构造函数与原型组合模式

\_\_proto\_\_ 属性:
原型对象创建出来的时候会有个默认的属性 \_\_proto\_\_
此属性指向原型对象自身.
对象如何访问原型?对象. \_\_proto\_\_
\_\_proto\_\_ 是一个非标准属性,为了保证通用性,不推荐在代码中使用,调试时使用.

constructor 属性:
原型对象创建出来的时候会有个默认的属性 constructor
此属性指向对应构造函数.
对象如何访问构造函数?对象.constructor.
重写构造函数的原型,constructor 属性也被重写,不再指向关联的构造函数需要在新替换的原型中手动添加 constructor 的指向

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

console.log(Person.prototype.constructor) //function Person(...){} 通过构造函数的原型访问构造函数

var p = new Person('zzw', '18')
console.log(p.name) //zzw

var p1 = new p.constructor('zjj', '18') // 通过已经创建好的实例对象调用构造函数
console.log(p1.name) // zjj

Person.prototype = {} // 重写构造函数的原型,constructor属性也被重写,不再指向关联的构造函数,而是指向function Object(){...},与构造函数没有关联了
console.log(Person.prototype.constructor) // function Object(){...}
```

所以,为了保证重写原型对象后依旧可以形成 构造函数-原型对象-实例对象 的闭环,需要做以下操作:

```js
Person.prototype = {
  constructor: Person //手动添加constructor的指向
}
```

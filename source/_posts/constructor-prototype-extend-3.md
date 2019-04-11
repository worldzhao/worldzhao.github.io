---
title: 构造函数、原型与继承（三）
date: 2017-07-08
tags: [js基础]
categories: js
---

![prototype.png](http://upload-images.jianshu.io/upload_images/4869616-de925ed1d446a5da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 原型链

- 定义: 每一个对象都存在构造函数,每一个构造函数都存在原型对象,每一个原型对象也存在构造函数,每一个原型对象的构造函数也存在原型对象,如此向上一层层查询,是一个链式结构,称为原型链.

## 继承

### 原型链继承:

利用原型中的成员可以被和其相关的对象所共享这一特性,可以实现原型继承通过修改原型链结构的方式实现继承,具体实现方式:**让子类的原型对象地址指向超类的实例对象,这样子类实例对象则会拥有超类实例对象的所有成员(包括超类实例成员与超类原型对象成员)**

```js
function SuperType() {
  this.class = '信管班'
}

function SubType(name) {
  this.name = name
}

var superObj = new SuperType()

SubType.prototype = superObj //继承

var subObj = new SubType('zzw')

console.log(subObj.class) //信管班
```

属性搜索原则:当访问一个对象的成员时,按照原型链依次搜索

### 利用原型链继承安全地拓展内置对象

利用原型链继承安全地拓展内置对象(倘若直接为内置对象添加成员,会影响整个开发团队)

```js
var arr = [1, 2, 3]
// 扩展内置对象(就是给内置对象新增成员)
Array.prototype.sayHello = function() {
  console.log('我是一个数组')
}
arr.sayHello()

// 如何安全地扩展一个内置对象? 继承内置对象,扩展自身,而不影响内置对象

function myArray() {}

var a = new Array()

myArray.prototype = a //继承数组对象

myArray.prototype.sayHello = function() {
  //扩展自身
  console.log('我是一个数组')
}

var myarr1 = new myArray()

myarr1.push(1, 2, 3, 4, 5)

console.log(myarr1)

myarr1.sayHello()
```

原型链式继承存在的问题:同原型模式存在的问题一样,超类构造函数的实例对象成为子类的原型对象,导致超类实例对象的对象成员变成了子类的原型对象成员,某个子类实例对象修改原型对象中的引用类型会影响所有实例对象.

示例代码如下:

```js
function SuperType() {
  this.colors = ['red', 'blue', 'black'] // 超类实例成员
}

SuperType.prototype.sayHello = function() {
  console.log('hello')
} // 超类原型成员

function SubType() {
  this.sex = 'male' // 子类实例成员;
}

SubType.prototype = new SuperType() // 继承
SubType.prototype.constructor = SubType // 保持闭三角关系

var subObj1 = new SubType() // 子类实例对象1
var subObj2 = new SubType() // 子类实例对象2
```

到此为止,我们就完成了一个经典的原型链式继承

可以看出子类实例对象 1 2 都拥有了 new SuperType()的成员,包括其 实例成员以及原型方法

```js
console.log(subObj1)
console.log(subObj2)
```

但存在一个问题,倘若修改子类实例对象 1 的 colors 会影响到实例对象 2,这也就是为什么很少单独使用原型链式继承的原因,就和为什么很少使用单独使用原型模式创建对象一样.

```js
subObj1.colors.push('white') //改变subObj1的colors
console.log(subObj2.colors) //影响到了subObj2的colors
```

### 组合式继承(原型链+构造函数)

就像解决原型模式一样,利用构造函数.在子类构造函数中调用超类的构造函数,就可以使得子类实例对象拥有自己的成员从而屏蔽原型对象中的对应成员.

```js
function SuperType() {
  this.colors = ['red', 'blue', 'black']
}

SuperType.prototype.sayHello = function() {
  console.log('hello')
}

function SubType() {
  SuperType.call(this) // 调用超类的构造方法，子类实例对象拥有了自己的colors 屏蔽了原型的colors
  this.sex = 'male'
}

SubType.prototype = new SuperType()
SubType.prototype.constructor = SubType

var subObj1 = new SubType()
var subObj2 = new SubType()

//此处相较于上文而言,子类构造函数中调用了超类的构造函数,子类实例对象于是拥有了自己的colors,屏蔽掉了new SuperType中的colors

console.log(subObj1)
console.log(subObj2)

subObj1.colors.push('white') // 改变subObj1的colors
console.log(subObj1.colors) // 影响到了subObj1的colors
console.log(subObj2.colors) // 并未影响到subObj2的colors
```

### 寄生组合式继承

组合继承把共享的属性、方法用原型链继承实现，独享的属性、方法用借用构造函数实现，所以组合继承几乎完美实现了 js 的继承。

为什么说是“几乎”？

组合继承有一个小 bug，实现的时候调用了两次超类构造函数，存在一点点性能问题。

```js
SuperType.call(this) // 第一次

SubType.prototype = new SuperType() // 第二次
```

怎么解决呢？于是“寄生继承”就出来了。

基本思路是：很明显，第二次调用构造函数是有一些多余的，因为超类构造函数中的成员已经通过 SuperType.call 给子类继承了，我们不必为了指定子类型的原型而调用超类型的构造函数[原型成员+实例成员(重复)]，需要的无非就是超类型原型的一个副本而已。

我们来实现一个 inheritPrototype 方法，通过它将子类的原型对象修改为继承自父类的原型对象的对象

```js
function inheritPrototype(subType, superType) {
  var prototype = Object.create(superType.prototype)
  prototype.constructor = subType
  subType.prototype = prototype
}
```

完整代码如下：

```js
function SuperType() {
  this.colors = ['red', 'blue', 'black']
}

SuperType.prototype.sayHello = function() {
  console.log('hello')
}

function SubType() {
  SuperType.call(this) // 调用超类的构造方法，子类实例对象拥有了自己的colors 屏蔽了原型的colors
  this.sex = 'male'
}

inheritPrototype(subType, superType)

var subObj1 = new SubType()
var subObj2 = new SubType()

//此处相较于上文而言,子类构造函数中调用了超类的构造函数,子类实例对象于是拥有了自己的colors,屏蔽掉了new SuperType中的colors

console.log(subObj1)
console.log(subObj2)

subObj1.colors.push('white') // 改变subObj1的colors
console.log(subObj1.colors) // 影响到了subObj1的colors
console.log(subObj2.colors) // 并未影响到subObj2的colors
```

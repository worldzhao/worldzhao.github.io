---
title: 函数
date: 2017-07-04
tags: [原生js]
categories: js
---
#### 函数

##### 创建函数的三种方式:

 * 函数声明(提升)

	function fun(){
		//函数体
	}

 * 函数表达式  

在使用函数表达式声明函数时,function后面可接函数名funcname 

但是在外部依旧通过name调用,funcname只限function内部访问,外部不可以

<!-- more -->

代码如下: 

	var name = function funcname(){
		
	}

 * Function构造函数法构造函数
 
 Function构造函数可以新建函数对象

    语法如下:
    Function函数所有参数都是字符串

    如果不传参数,表示创建一个空函数

    var 函数名 = new Function();

    如果只有一个参数那么必须是函数体

    var 函数名 = new Function('函数体');

    如果传多个参数,那么最后一个参数为函数体,前面的参数为将要创建的函数的形参名

    var func=new Function('参数1','参数2',...,'函数体');

    1. Function也可以被当作一个构造函数,通过Function new出来的函数可以被当作是实例化的对象
    2. Function这个构造函数也有原型对象
    3. Function.prototype是一个空的函数(对象),是由Object构造函数创建的一个实例
    4. Function.prototype.__proto__是Object.prototype,即Function的原型对象的原型对象其实就是所有obj的原型对象  
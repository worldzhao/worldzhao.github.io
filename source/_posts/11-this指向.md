---
title: this指向问题
date: 2017-07-04
tags: [原生js]
categories: js
---
#### 首先我们需要复习函数的前三种调用模式

##### 函数调用模式

函数有4种调用模式:
        1. 方法调用模式(对象)
        2. 函数调用模式(window对象)
        3. 构造器调用模式(new)
        4. 上下文模式
了解函数调用模式的不同,对于我们理解函数内部this指向有着重要的作用

<!-- more -->

###### 方法调用模式与函数调用模式
demo1:

        
        var age=38;
        var obj={
            age:18,
            getAge:function(){
                return this.age;
            }
        }
        console.log(obj.getAge());//18  方法调用模式

        var getAge=obj.getAge;
        console.log(getAge());//38   函数调用模式

demo2:

        var age=38;
        var obj={
            age:18,
            getAge:function(){//方法调用 this指向调用对象
                console(this.age);
                function foo(){
                    console.log(this.age);
                }
                foo(); //函数调用,this指向window对象(非严格模式)
            }
        }
      
      obj.getAge();// 18  38 


demo3:

        var length=10;
        function fn(){
            console.log(this.length);
        }

        var obj={
            length:5,
            method:function(fn){
                fn();//10
                arguments[0]();//2? 
            }
        }
        obj.method(fn,123);//10 2

>  相当于arguments调用了fn 返回arguments的长度
>  
>arguments是一个包含很多属性的对象,键名为索引,键值为传进来的参数值

>获取对象的属性值有两种方法:点方法与中括号法 obj.name === obj['name']

>arguments[0] === arguments.0(并不能这样写)

>调用arguments.0\(\)==arguments\[0\]\(\)  实际上是方法调用

>理解不了看下面:

    var obj={abc:fn};
    obj.abc();//方法调用
    obj['abc']();//方法调用

###### 构造器调用模式:
demo4:

        function Person(name,age){
            this.name=name;
            this.age=age;
        }
        var p=new Person('zzw','18');

>new关键字就做了一点微小的工作
        var obj={};
        obj.\_\_proto\_\_=Person.prototype;
        Person.call(obj);//通过call方法强行将构造函数内部的this指向了obj

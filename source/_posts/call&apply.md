---
title: call与apply
date: 2017-07-04
tags: [原生js]
categories: js
---
#### call与apply
首先来看一个demo:

        demo1:

            var p1={
                name:"赵志文",
                age:"18"
            }

            var name="周娇娇";

            function getName(){
                console.log(this.name,this);
            }

            getName();//周娇娇 window //函数调用,this指向window(非严格模式)

            getName.call(p1);//赵志文 Object //改变了getName内部的this指向,指向p1

            getName.apply(p1);//同上

<!-- more -->

相信各位读者通过上面的demo能够很轻易的知道call与apply的作用:改变调用函数内部的this指向.

我们回顾一下之前的构造函数:

    demo2:        
            function Person(name,job){
                //默认隐含的操作,将new创建的对象赋值给this
                this.name=name;
                this.job=job;
                this.sayHello=function(){
                    console.log('hello');
                }
            }

            var winter = new Person("winter","coding");

构造函数的执行过程: 

* 使用new关键字创建对象;
* 调用构造函数,将新创建出来的对象赋值给对象函数内部的this;//如何做到?
* 在构造函数内部使用this为新创建的对象新增成员;
* 默认返回新创建的对象,普通函数如果不写返回语句,会返回undefined.

> new关键字就做了一点微小的工作
        var obj={};
        obj.\_\_proto\_\_=Person.prototype;
        Person.call(obj);

此处也是call与apply的一个典型用法.

**使用call与apply,可以修改函数调用上下文,也就是this的值,这两个方法都是定义在Function.prototype中,所有函数都可以调用**

##### Function.prototype.apply()/call()

MDN文档中写道:"apply()方法在指定this值和参数(参数以数组或类数组形式存在)的情况下调用某个函数."

>apply()方法与call()方法仅有的区别:call()方法接收的是一个参数列表,而apply()方法接收的是一个包含多个参数的数组(或类数组对象);

语法:
    
        fun.apply(thisArg [,argsArray]);
        fun.call(thisArg,para1,para2,...);
来看一个小demo:

    demo3:

        function demo3(a,b){
            console.log(this.name+"吃了"+(a+b)+"个西瓜");
        }

        var name="zzw";

        demo3(1,2);//函数调用模式 非严格模式下指向window zzw吃了3个西瓜

        var obj={
            name:'zjj'
        }

        var obj1={
            name:'sb'
        }
        
        demo3.call(obj,4,5);//改变this指向为obj同时传参  zjj吃了9个西瓜
        demo3.apply(obj1,[100,99]);//改变this指向为obj同时传参 sb吃了199个西瓜

##### 案例:

* 求一个数组中的最大值

>js中Math对象有个方法max,求出参数中的最大值
但是往往给定的是数组 ,怎么办捏
使用apply,这里就不改变this了,单纯数组转参数

知识点:apply与call第一个参数为null时,单纯调用函数,着眼于参数类型(例如给定数组,通过apply数组转参数列表)

<pre>
    var arr=[1,20,157,76,84];
    var max=Math.max.apply(null,arr);
    console.log(max);
</pre>

* 将传入函数的参数打印出来,用'-'连接

法一:自己的傻瓜拼接字符串法

    function demo4(){
        var res="";
        for(var j=0;j<arguments.length;j++){
            if(j<arguments.length-1){
                res+=arguments[j]+"-";
            }else{
                res+=arguments[j];
            }
        }
        console.log(res);
    }
    demo4(1,2,3,"abc","zzw");//1-2-3-abc-zzw

法二:可以重新创建一个数组容纳参数,调用join方法

    function demo4(){
        var arr=[];
        for (var i = arguments.length - 1; i >= 0; i--) {
            arr.unshift(arguments[i]);
        }
        arr=arr.join('-');
        console.log(arr);
    }
    demo4(1,2,3,"abc","zzw");//1-2-3-abc-zzw


法三:直接借用join方法,改变join的内部this指向

    function demo4(){
       //return arguments.join("-");//报错 伪数组并非数组 不可以使用join
       var str=Array.prototype.join.call(arguments,"-");//改变join内部的this
       console.log(str);
       //或写成 var str = [].join.call(arguments,"-");
    }
    demo4(1,2,3,"abc","zzw");//1-2-3-abc-zzw
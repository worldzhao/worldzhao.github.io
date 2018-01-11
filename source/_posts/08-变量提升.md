---
title: 变量提升
date: 2017-07-04
tags: [原生js]
categories: js
---
#### 变量提升

JS代码的执行分为两个阶段
**预解析阶段**

   * 变量名和函数声明提升至当前作用域的最上方(只提升声明,不提升赋值)

**执行阶段**

   * 按照从上至下的顺序执行语句 

注意:

* 变量与函数同名时,只提升函数,不提升变量 
* 函数同名时,都提升,后面的函数覆盖前面的函数
* 函数表达式只会提升变量名,赋值语句在执行阶段按照顺序执行
* 变量提升是分块的<script></script>
* 条件是函数声明能否被提升取决于浏览器,不推荐使用if(true){function(){}}; 

<!-- more -->
实例:

    //js中没有块级作用域的说法,大括号并不会创建一个作用域

    for(var i=0;i<5;i++){
        console.log(i);//0 1 2 3 4
    }
    console.log(i);//5 for循环外部依旧可以访问到i


只有函数能够创建作用域

变量提升是分作用域的

函数与变量同名只提升函数  但是变量赋值还是要执行的

代码:

    var num=10;
    function num(){
        console.log(123);
    }
    num();

预编译

    function num(){
        console.log(123);
    }
    num=10;
    num();//is not a function


函数与函数同名全部都会提升,但会以函数声明顺序进行覆盖,后面覆盖前面的

代码:

    function f1(){
        console.log('first');
    }
    function f1(){
        console.log('last');
    }

预编译:

    function f1(){
        console.log('last')
    }


函数表达式与变量提升方式相同,变量赋值不会被提升,但变量声明会提升
 
代码:

    f1()//undefined
    var f1=function(){
        console.log('f1');
    }

预编译:

    var f1;//提升
    f1();//调用 undefined
    f1=function(){//赋值
        console.log('f1');
    }

代码:

    function foo() {
        var num = 123;
        console.log(num); //123
    }
    foo();
    console.log(num); //is not defined

预编译:

    function foo() {
        var num;
        num = 123;
        console.log(num); //?123
    }
    foo();
    console.log(num); //is not defined

    //is not defined 没有定义
    //undefined  定义了没有赋值

代码:

    var scope = "global";
    foo();
    function foo() {
        console.log(scope); //undefined
        var scope = "local";
        console.log(scope); //local
    }

预编译:

    var scope;
    function foo(){
        var scope;
        console.log(scope); 
        scope = "local";
        console.log(scope); 
    }
    scope = "global";
    foo();

    //in 关键字 判断某个对象中是否有某个属性

代码:

    function f1(){
       if("a" in window){
           var a = 10;
       }
       alert(a);//undefined
    }
    f1();

预编译:

    function f1(){
       var a;
       if("a" in window){
           a = 10;
       }
       alert(a);//undefined
    }
    f1();

代码:

    if("a" in window){
        var a = 10;
    }
    alert(a); //10


预解析:

    var a;
    if("a" in window){
        a = 10;
    }
    alert(a);

代码:

    if(!"a" in window){
        var a = 10;
    }
    alert(a); // undefined


预解析:

    var a;
    if(!"a" in window){
      a=10;
    }
    alert(a);

代码:

    var foo = 1;
    function bar() {
        if(!foo) {//此处提升了foo但是是undefined 判断语句会转为!undefined -->true
            var foo = 10;
        }
        alert(foo); //10
    }
    bar();

预解析:

    var foo;
    function bar(){
        var foo;//undefined
        if(!foo) {//!undefined --> ture
            foo = 10;//10
        }
        alert(foo); //10
    }
    foo = 1;
    bar();

代码:

    function Foo() {
        getName = function(){ alert(1); };
        return this;
    }

    Foo.getName = function() { alert(2); };

    Foo.prototype.getName = function(){ alert(3); };

    var getName = function() { alert(4); };

    function getName(){ alert(5); }

    Foo.getName(); //2

    getName(); //4
    
    Foo().getName(); //1
      
    getName(); //1
    
    new Foo.getName(); //2
    
    new Foo().getName(); //3
    
    new new Foo().getName(); //3

预解析:

    function Foo() {
        getName = function(){ alert(1); };
        return this;
    }

    var getName;

    function getName(){ alert(5); }

    Foo.getName = function() { alert(2); };

    Foo.prototype.getName = function(){ alert(3); };

    getName = function() { alert(4); };

   

    getName(); //4
    
    Foo().getName(); //1
      
    getName(); //1

    Foo.getName(); //2
    
    new Foo.getName();//2 先执行Foo.getName()  new关键字无效

    (new Foo).getName();//3 先执行new Foo创建对象
    
    new Foo().getName(); //3 先执行new Foo()创建对象
   
    new new Foo().getName(); //3 相当于 new Foo().getName() new关键字无效
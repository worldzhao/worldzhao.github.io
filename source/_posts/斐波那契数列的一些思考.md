---
title: 关于斐波那契数列的一些思考
date: 2017-09-27
tags: [原生js]
categories: js
---

先看代码:
```js
function fib(n){
    if(n===1){
        return 1;
    }
    if(n===2){
        return 1;
    }
    if(n>2){
        return fib(n-1)+fib(n-2);
    }
}
console.log(fib(11));
```


这是一个很典型的利用递归计算斐波那契数列

递归的缺点也是显而易见的,我们计算fib(6)时 要计算fib(5)和fib(4)

而后计算fib(7)时,又要重复计算fib(6)与fib(5)

很明显,我们之前已经计算过了fib(6)与fib(5),现在重复计算,造成了浪费。


首先我们来观察一下 当n从20到21时,调用此函数 内部会多调用多少次此函数?

```js
var count=0;//计数器
function fib(n){
    count++;
    if(n===1){
        return 1;
    }
    if(n===2){
        return 1;
    }
    if(n>2){
        return fib(n-1)+fib(n-2);
    }
}
console.log(fib(20));
console.log(count);//13529次
count = 0;
console.log(fib(21));
console.log(count);//35420次  从20到21 调用次数增加很多
```
其实我们完全可以将之前计算过的数值用一个数组保存起来，如果需要重复计算，先去数组内部查找，如果数组里面存在该结果，直接return

```js
var cache = [0,1,1];
function fib(n){
    if(n<=2){
        return cache[n];
    }else{
        if(cache[n]){ //如果缓存数组中存在 直接返回
            return cache[n];
        }else{
            var temp = fib(n-1)+fib(n-2); //如果缓存数组中不存在 进行递归
            cache[n]=temp;   //将递归结果存入缓存数组
            return temp;
        }
    }
 }

 console.log(fib(20));
```
这样已经能够节省很多递归造成的空间浪费。

但缓存数组孤零零的放在全局作用域，不够安全，封装性也不好。

我们希望他们联系的能够更紧密一些，就像一个整体。

于是有了下面的代码：
```js
var fib = (function(){
    var cache = [0,1,1];
    var fib = function() {
        if(n<=2){
            return cache[n];
        }else{
            if(cache[n]){ //如果缓存数组中存在 直接返回
                return cache[n];
            }else{
                var temp = fib(n-1)+fib(n-2); //如果缓存数组中不存在 进行递归
                cache[n]=temp;   //将递归结果存入缓存数组
                return temp;
            }
        }
    }
    return fib;
})()
```
这样，我们通过闭包，只能通过返回的fib方法对cache进行操作了。

当然，你也可以像下面这样写：

```js
function createCache(){
    var cache=[0,1,1];//缓存数组被封装在闭包中,外界只能通过返回的方法进行操作
    return function editCache(index,value){
        if(value==undefined){
            return cache[index];
        }else{
            cache[index]=value;
        }
    }
 }

 var fibCache=createCache();//创建缓存数组并且获取接口方法

 function fib(n){
    if(n<=2){
        return fibCache(n);
    }else{
        if(fibCache(n)){
            return fibCache(n);
        }else{
            var temp = fib(n-1)+fib(n-2);
            fibCache(n,temp);
            return temp;
        }
    }
 }
```
递归效率低是函数调用的开销导致的。
在一个函数调用之前需要做许多工作，比如准备函数内局部变量使用的空间、搞定函数的参数等等，这些事情每次调用函数都需要做，因此会产生额外开销导致递归效率偏低，[知乎](https://www.zhihu.com/question/35255112/answer/62180021)
其实一般递归的方法我们都可以通过迭代的方式来做，for循环就是一个很好的选择：

```js
function fib(n) {
    if (n == 1 || n == 2) {
        return 1;
    }
    var arr = [0,1,1];
    for(var i = 2; i <= n; i++) {
        arr[i] = arr[i-1] + arr[i-2];
    }
    return arr[n];
}
```

看起来也挺好的，是吧。

下面进入算法时间，题目来自《剑指Offer》，当然，递归依然是主角。

题目一：
一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级。求该青蛙跳上一个 n 级的台阶总共有多少种跳法？

解题：数学归纳法。
1级台阶，1种跳法，直接跳上1级台阶 f(1) = 1
2级台阶，2种跳法，直接跳上2级台阶或者连续跳两次，每次一级 f(2) = 2
3级台阶，3种跳法，如果第一次跳1级，剩下2级台阶，f(2) = 2种跳法;如果第一次跳2级，剩下1级台阶，f(1) = 1种跳法。f(3) = f(2) + f(1) = 2 + 1 = 3
4级台阶，5种跳法，如果第一次跳1级，剩下3级台阶，由上一条可知有3种跳法；如果第一次跳2级，剩下2级台阶，由上上条可知有2种跳法，f(4) = f(3) + f(2) = 3 + 2 = 5
规律很清楚了，斐波那契数列变一变就是的了：

```js
function jump(n) {
    if (n <= 0) {
        return;
    } else if (n > 0 && n <= 2) {
        return n;
    } else if (n > 2) {
        return jump(n-1) + jump(n-2);
    }
}
```

题目二：
一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级……它也可以跳上 n 级。求该青蛙跳上一个 n 级的台阶总共有多少种跳法？

解题：继续归纳。
1级台阶，1种跳法
2级台阶，2种跳法
3级台阶，4种跳法 f(3) = f(2) + f(1) + 1 = 4
4级台阶，第一次跳1级，后面有f(3)种跳法，第一次跳2级，后面有f(2)种跳法，第一次跳3级，后面有f(1)种跳法，第一次跳4级，没了。
总共f(4) = f(3) + f(2) + f(1) + 1 = 8种跳法
5级台阶，f(5) = f(4) + f(3) + f(2) + f(1) + 1 = 16种跳法
归纳得，f(n) = f(n-1) + f(n-2) + ... + f(1) + 1

```js
function jump(n) {
    if (n <= 0) {
        return;
    } else if (n > 0 && n <= 2) {
        return n;
    } else if (n > 2) {
        var tmp = 0;
        while(number > 1){
            tmp+=jump(n-1);
            number--;
        }
        return tmp+1;
    }
}
```


再来看看jQuery中的缓存机制:

自己实现如下:

    function createCache(){//不使用自执行函数创建块级作用域的原因是:
        //我们要多次创建这里面的内容(有多个缓存),而不是像原来一样仅仅创建一次

        var cache={}; //创建缓存对象

        var index=[];//存放键名的数组,用于缓存过多时进行清理(因为缓存对象无法判断有多少个缓存在内)

        return function editCache(key,value){

            if(value===undefined){

                return cache[key];//如果不传value就是查询

            }else{

                cache[key]=value;//如果传了value就是设置

                index.push(key);

                if(index.length>=50){//当缓存的数量到达一个临界时(此处是50),删除最早的缓存

                    var tempKey = index.shift();//获取键名  并 删除 index中的该键

                    delete cache[tempKey];//删除cache内的属性

                }
            }
        } 
    }

    var eleCache=createCache();

    var typeCache=createCache();//多次创建,每一个Cache都有自己的一个空间

    var classCache=createCache();
    
    var eventCache=createCache();

    eleCache("name","zhaozhiwen");//store

    elemCache("name")//get

jQuery源码:

    function createCache() {
        var keys = [];
        function cache( key, value ) {

            //这里直接将这个函数当作缓存对象,减少了创建对象的次数

            //同时由于缓存属性都是直接加在这个对象上  且return出去了 可以直接通过cache['键名']获取缓存值 于是函数内部设置缓存即可

            //分两个角度看:  当cache是对象是 他有缓存属性 用于查询
            //当cache是方法时 他给自己(对象)设置添加缓存

            //更简洁 jQuery的优美之处啊 巧夺天工

            // 使用(key + " ") 是为了避免和原生（本地）的原型中的属性冲突
            if ( keys.push( key + " " ) > 3 ) {
                // 只保留最新存入的数据
                delete cache[ keys.shift() ];
            }
            // 1 给 cache 赋值
            // 2 把值返回
            return (cache[ key + " " ] = value);
        }
        return cache;
    }

    var typeCache = createCache(); //创建缓存对象并获取接口方法

    /*
    typeCache("monitor","周娇娇");
    console.log(typeCache["monitor"]);//这样是查不到的,因为存储的时候 加了" "
    */

    typeCache("monitor1","张学友");  //向缓存中存入内容
    console.log(typeCache["monitor1 "]); //通过键名取出内容

    typeCache("monitor2","刘德华");
    console.log(typeCache["monitor2 "]);

    typeCache("monitor3","彭于晏");
    console.log(typeCache["monitor3 "]);

    typeCache("monitor4","赵志文"); //这次再进行缓存,超出了限制,第一个缓存被干掉了

    console.log(typeCache["monitor1 "]); //undefined

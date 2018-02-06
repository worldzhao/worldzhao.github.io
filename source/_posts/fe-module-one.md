---
title: 前端模块化（一）
date: 2017-09-19
tags: [原生js]
categories: js
---
其实作为一个秋招狗，前端还没写够一年就来大谈特谈这些历史性问题，的确是不够格的，但是这又是学习前端学习者无法避免的一个问题，加上面试提问颇多，于是来总结一二。

>站在巨人的肩膀上。
参考资料：
[Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)
[Javascript模块化历程](http://web.jobbole.com/83761/)

# 模块

模块就是实现特定功能的一组方法。

最开始笔者学习js的时候，新建一个js文件，直接在里面写逻辑代码，然后script标签引入即可，一两个小网页是无妨的，但是随后后面逻辑越来越复杂，代码量越来越大，可复用的要求越来越多，笔者也是看着越来越混乱的js文件焦灼无比。

毕竟js最初只是作为一门“玩具语言”闯进开发者的视野，甚至这门语言仅仅花了不到一个月的时间就被创造了出来，我们如今看见的js，是开发者与互联网不断的磨合妥协下的js.

所以开发者不得不使用软件工程的方法，管理网页的业务逻辑。

但是，Javascript不是一种模块化编程语言，它不支持"类"（class），更遑论"模块"（module）了。（正在制定中的ECMAScript标准第六版，将正式支持"类"和"模块"，但还需要一段时间才能投入实用。）

“Don`t repeat yourself”是我们的准则，如何更好的去实践这条准则就是我们今天要讲的内容。

## 函数封装

直接在页面中写逻辑代码是我最初的做法，后来慢慢地学会了将重复的代码封装至一个函数之内，需要时调用函数即可。函数一个功能就是实现特定逻辑的一组语句打包，而且JavaScript的作用域就是基于函数的，所以把函数作为模块化的第一步是很自然的事情。

只要把不同的函数（以及记录状态的变量）简单地放在一起，就算是一个模块。

```js
function fn1(){
    statement
}
function fn2(){
    statement
}
```

这样在需要的以后加载函数所在文件，调用函数就可以了。

这种做法的缺点很明显：污染了全局变量，无法保证不与其他模块发生变量名冲突，而且模块成员之间没什么关系。

## 对象

我们需要一个方法将全局变量减少，于是对象写法很自然的就出现了。

```js
var myModule = {
    fn1: function(){
        statement
    },
    fn2: function(val){
       statement
    }
}
```

世界美好了那么一点点。

这样我们在希望调用模块的时候引用对应文件，然后

```js
myModule.fn1();
```

但是我们看看下面这段代码。

```js
var myModule = {
    var1: 1,
    var2: 2,
    fn1: function(){
        return var1;
    },
    fn2: function(val){
        var2 = val;
    }
}
```

不知道大家发现没有，我明明可以直接通过

```js
myModule.var1;
```

直接获取var1，那我还要调用fn1去return干嘛，简直奇怪。

没错，看似不错的解决方案，但是也有缺陷，这样的写法会暴露所有模块成员，内部状态可以被外部改写，毫无封装型可言。

在别的语言中可以很轻易的实现私有变量，例如java的private关键字，但是js需要通过别的手段来实现。

我们希望外部不能够如此随意的访问或者修改某一个变量，只能通过我们规定的特权方法去访问模块的变量，而无法直接改写内部状态，于是有了闭包。

## 闭包

没错，闭包又出现了，当初学习闭包的时候，网上的种种文章，描述了闭包的定义，闭包的特点，以及如何实现闭包，但是，他们就是没有告诉你闭包主要是拿来做什么的，在我看来，闭包就是模块化的基础！

前面我们提到过JavaScript的作用域就是基于函数的，我们可以通过这一点将想要隐藏起来的变量置于函数作用域中，根据作用域链访问的原则，外层是无法访问到该变量的，

看下面代码
```
var module1 = (function(){
　　　　var var1 = 0;
　　　　var fn1 = function(){
　　　　　　//...
　　　　};
　　　　var fn2 = function(){
　　　　　　//...
　　　　};
　　　　return {
　　　　　　fn1 : fn1,
　　　　　　fn2 : fn2
　　　　};
　　})();
```

这里是一个立即执行函数，return一个对象，该对象内部的函数是可以直接访问到var1的（闭包），而外部再也不能轻易的去访问var1了，只能通过module1的方法去访问，世界又美好了一点。

PS：我一直觉得立即执行函数的本质就只是为了立即执行而已，不理解为什么还特地产生了一个概念，如果您有更好的解答，请写下您的评论。

module1就是Javascript模块的基本写法。

当然还有类似于jq的那种写法，

```js
var module1 = (function ($, YAHOO) {
　　　　//...
　　})(jQuery, YAHOO);
```

独立性是模块的重要特点，模块内部最好不与程序的其他部分直接交互。

为了在模块内部调用全局变量，必须显式地将其他变量输入模块。

输入全局变量，这样做除了保证模块的独立性，还使得模块之间的依赖关系变得明显。

看懂了上面的，这一个应当很容易理解。

下集预告：AMD/CMD/Commonjs/ES6

最近一直在总结校招重点，而那些重点基本是搜集别人的文章里面的，发在简书上未免有拿别人的文章骗赞之嫌。

所以有面试的小伙伴们可以去我的博客看看，相互交流，共同提升。
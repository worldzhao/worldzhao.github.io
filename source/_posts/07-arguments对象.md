---
title: arguments对象
date: 2017-07-04
tags: [原生js]
categories: js
---
#### arguments对象

属性:
* arguments.callee
指向函数本身,常见于递归

实例:实现一个计算斐波那契数列的递归函数

    //常见写法
    function feibonaqi(n){
        if(n===1){
            return 1;
        }
        if(n===2){
            return 1;
        }
        if(n>2){
            return feibonaqi(n-1)+feibonaqi(n-2);
        }
    }
    console.log(feibonaqi(11));
    
<!-- more -->

    //倘若清除函数指向则会出现问题
    function feibonaqi(n){
        if(n===1){
            return 1;
        }
        if(n===2){
            return 1;
        }
        if(n>2){
            return feibonaqi(n-1)+feibonaqi(n-2);
        }
    }
    var myfeibonaqi = feibonaqi;
    feibonaqi=null;//清除原函数的指向,导致递归中调用失败
    console.log(myfeibonaqi(11));//报错



    //使用arguments.callee,指向函数自身 而不通过函数名指向,解决上述问题
    function feibonaqi(n){
        if(n===1){
            return 1;
        }
        if(n===2){
            return 1;
        }
        if(n>2){
            return arguments.callee(n-1)+arguments.callee(n-2);
        }
    }

    var myfeibonaqi = feibonaqi;
    feibonaqi=null;//清除原函数的指向,递归中却并没有使用原函数名
    console.log(myfeibonaqi(11));//成功


* arguments.length
传入参数的个数

1. 一个函数有形参的时候可以不传参
2. 一个函数没有形参的时候可以传参
3. 以上两条规则能够实现都是因为函数内部存在了arguments对象
4. 一个函数不论有无形参,调用的时候都会将实参值传入arguments对象
5. arguments通过下标的形式取出参数值,但是arguments并不是一个数组
6. 通过arguments的特性可以实现方法重载

实例:数组去重
方法一:

    function noRepeat(){
        var arr=[];
        for(var k=0;k<arguments.length;k++){
            if(arr.indexOf(arguments[k]!=-1)){
                 arr.push(arguments[k]);
            }           
        }
        return arr;
    }
    console.log(noRepeat(1,1,2,2,3,3));//1,2,3

方法二:

    function noRepeat(){
        var arr=[];
        for(var k=0;k<arguments.length;k++){
            arr[k]=arguments[k];
        }
        for(var i=0;i<arr.length;i++){
            for(var j=i+1;j<arr.length;j++){
                if(arr[i]===arr[j]){
                    arr.splice(j,1);
                }
            }
        }
        return arr;
     }
     console.log(noRepeat(1,1,2,2,3,3));//1,2,3


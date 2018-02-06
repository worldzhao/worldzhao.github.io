---
title: 数组去重
date: 2017-07-05
tags: [原生js]
categories: js
---
### 前言：

在做音乐播放器项目时，有一个功能是添加歌曲，很明显，歌曲是不能重复添加的。这时候我想到了es6的Array.from(new Set(arr))去重方法，很明显是不行的，歌曲都是对象，大部分去重方法只能解决基本数据类型的去重，犹记得学java时的equal方法，比较两个对象是否相等着实折腾了以下当初的我，这篇博文主要是总结一下js去重。


* 方法一：利用ES6的Array.from()/扩展运算符 以及 Set

    >Array.from(): The Array.from() method creates a new Array instance from an array-like or iterable object.

    该方法接收两个参数要转换的非数组对象,对每个元素进行处理的方法（可选）.在js中，有很多类数组对象（array-like object）和可遍历（iterable）对象（包括ES6新增的数据结构Set和Map），常见的类数组对象包括document.querySelectorAll()取到的NodeList，以及函数内部的arguments对象。它们都可以通过Array.from()转换为真正的数组，从而使用数组的方法。事实上只要对象具有length属性，就可以通过Array.from()转换为真正的数组。

    >Set:A collection of unique values that may be of any type.

    Set:一个可以是任何类型的独一无二的值的集合.

```js
function unique(arr){
    return Array.from(new Set(arr));
}
```   

    你也可以这样写:

```js
function unique(arr){
    return [...new Set(arr)];
}
```   

*  方法二：遍历数组，建立新数组，利用indexOf判断是否存在于新数组中，不存在则push到新数组，最后返回新数组

    >Determines the index of the specific IThing in the list.

    indexOf() :方法可返回某个指定的字符串值在字符串中首次出现的位置。如果没有则返回-1

```js
function unique(arr){
    var newArr = [];
    for(var i in arr) {
        if(newArr.indexOf(arr[i]) == -1) {
            newArr.push(arr[i])
        }
    }
    return newArr;
}
```   


*  方法三：遍历数组，利用object对象的key值保存数组值，判断数组值是否已经保存在object中，未保存则push到新数组并用object[arrayItem]=true的方式记录保存.

```js
function unique(arr) {  
    let hashTable = {};
    let newArr = [];
    for(let i=0,l=arr.length;i<l;i++) {
        if(!hashTable[arr[i]]) {
            hashTable[arr[i]] = true;
            newArr.push(arr[i]);
        }
    }
    return newArr;
}
```   


* 方法四：先排序，新数组最后一项为旧数组第一项，每次插入判断新数组最后一项是否与插入项相等

```js
function unique(arr) {
    var newArr = [];
    var end; //end其实就是一道卡
    arr.sort();
    end = arr[0];
    newArr.push(arr[0]);
    for (var i = 1; i < arr.length; i++) {
        if (arr[i] != end) {
            newArr.push(arr[i]);
            end = arr[i]; //更新end
        }
    }
    return newArr;
}
```   

* 方法五：使用indexOf找到的是第一个出现的index的特点

```js
function unique (arr) {
    var res = arr.filter(function (item, index, array) {
        return array.indexOf(item) === index;
    })
    return res;
}
```
以上五种方法都是对于基本数据类型而言，如果换做对象数组就无能为力了，下面是对象数组的去重方法


*  方法一：利用对象的键名不能重复的特点

```js
    function unique(arr){
        let unique = {};
        arr.forEach(function(item){
        unique[JSON.stringify(item)]=item;//键名不会重复
        })
        arr = Object.keys(unique).map(function(u){ 
        //Object.keys()返回对象的所有键值组成的数组，map方法是一个遍历方法，返回遍历结果组成的数组.将unique对象的键名还原成对象数组
        return JSON.parse(u);
        })
        return arr;
    }
```

```js
    /*map方法使用示例:*/
    var map = Array.prototype.map
    var a = map.call("Hello World", function(x) { return x.charCodeAt(0); })
    // a的值为[72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]
```
   存在的问题:
    {x:1,y:2}与{y:2,x:1}通过JSON.stringify字符串化值不同，但显然他们是重复的对象.


*  方法二：还是利用对象的键名无法重复的特点,必须知道至少一个对象数组中的对象的属性名

```js
var songs = [
        {name:"羽根",artist:"air"},
        {name:"羽根",artist:"air"},
        {name:"晴天",artist:"周杰伦"},
        {name:"晴天",artist:"周杰伦"},
        {artist:"周杰伦",name:"晴天"}
    ];

function unique(songs){
    let result = {};
    let finalResult=[];
    for(let i=0;i<songs.length;i++){
        result[songs[i].name]=songs[i];
        //因为songs[i].name不能重复,达到去重效果,且这里必须知晓"name"或是其他键名
    }
    //console.log(result);{"羽根":{name:"羽根",artist:"air"},"晴天":{name:"晴天",artist:"周杰伦"}}
    //现在result内部都是不重复的对象了，只需要将其键值取出来转为数组即可
    for(item in result){
        finalResult.push(result[item]);
    }
    //console.log(finalResult);[{name:"羽根",artist:"air"},{name:"晴天",artist:"周杰伦"}]
    return finalResult;
}

console.log(unique(songs));
```


![原数组（重复）.png](http://upload-images.jianshu.io/upload_images/4869616-345ec1c3f2d06e43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![result.png](http://upload-images.jianshu.io/upload_images/4869616-0981489f9d3cf515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![finalResult.png](http://upload-images.jianshu.io/upload_images/4869616-a1daf130aac7ddeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


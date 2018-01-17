---
title: require('./expample.js).default详解
date: 2018-01-17
tags: [原生js]
categories: js
---
# require('./expample.js).default详解

最近总碰到类似于
```js
var a = require('./expample.js).default
```
这样的代码，感觉很奇葩，总结一波。

为什么会出现这个问题?

`import` 是静态编译的，而 `require` 可以动态加载，也就是说你可以通过判断条件来决定什么时候去 `require` ，而 `import` 则不行，所以有时候我们会面临需要通过`require` 去导入一个es6模块（比如react-hot-loader官方demo :P）

当然，这只是场景之一。

## 前置知识
1. ES6 Module常用语法。譬如`export`导出模块接口 | `import` 倒入模块| `export default`语法糖
2. Node.js模块常用。譬如`module.exports` | `require`
3. ES6模块与commonjs模块的区别（静态编译与动态加载 | 值得引用与值的拷贝）

如果上述前置知识您有所不了解的话，建议拜读一下阮一峰老师的《ECMAScript 6 入门》一书中的两个章节：

1. [Module 的语法](http://es6.ruanyifeng.com/#docs/module)
2. [Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)

如果您已经具备了上述知识，我们来讨论一下

1. `export default`为什么是语法糖
2. require一个ES6 Module

## `export default`为什么是语法糖

**`default`关键字 说白了，就是别名(as)的语法糖**

以下代码应当是非常常见的：

导出接口
```js
// a.js
function a(){}
export {a}
```
导入模块
```js
// b.js
import {a} from './a'
```
花括号就是解构赋值的语法，我们可以理解为`export`导出了一个对象，对象里存在a这个函数，就像下面这样
```js
{
  a:function(){}
}
```

于是就有了后面的通过结构赋值取出a，所以变量名必须一致。

> ECMAScript 6 入门：从前面的例子可以看出，使用import命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。<br>为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到export default命令，为模块指定默认输出。

`default`可以理解为这一语法的语法糖

导出接口
```js
// d.js
export default function() {}

// 等效于：
function a() {};
export {a as default};
```
导入模块
```js
import a from './d';

// 等效于，或者说就是下面这种写法的简写，是同一个意思
import {default as a} from './d';
``` 
这个语法糖的好处就是import的时候，可以省去花括号{}。

简单的说，如果import的时候，你发现某个变量没有花括号括起来（没有*号），那么你在脑海中应该把它还原成有花括号的as语法。

本质上依旧是结构赋值呀，只不过我们写的更为简便，假装花括号消失了罢了。

## 如何require一个ES6 Module

stackoverflow上有一个针对本文题目比较好的回答，原文如下：

>Finally, the require and require.default... when dealing with ES6 imports (export default MyComponent), the exported module is of the format {"default" : MyComponent}. The import statement correctly handles this assignment for you, however, you have to do the require("./mycomponent").default conversion yourself. The HMR interface code cannot use import as it doesn't work inline. If you want to avoid that, use module.exports instead of export default.


我来翻译下[原文](https://stackoverflow.com/questions/43247696/javascript-require-vs-require-default)：

最后，require和require.default...当在node中处理ES6 模块(export default mycomponent)导入的时候，导出的模块格式为
```js
{
  "default": mycomponent
}
```
`import` 语句正确地为你处理了这个问题，然而你必须自己执行 `require("./mycomponent").default`. HMR(热更新模块)不在 `inline` 模式工作的情况下，接口代码不能使用 `import` ，如果你想避免，使用`module.exports`而不是`export default`;


上文提到过，`export` 关键字是导出一个对象，对象内存在一个属性(我们要暴露的)，`export default` 则是 `export` 语法糖，`import` 一个`export default` 暴露出来的模块包含了解构赋值的步骤，所以在node中使用`require`去请求一个`export default`的模块需要我们通过`.`语法去取出对象中的属性(因为require木有解构赋值)，清晰明了。

换个说法，如果 `require` 的 commonjs规范的模块，即：
导出：
```js
// a.js
module.exports = {
  a:'helloworld'
}
```
导入：
```js
// b.js
var m = require('./a.js');
console.log(m.a); // helloworld
```
这样就显得非常清晰，我们 `module.exports` 的是啥，`require` 的就是啥。

但export default包装了一层语法糖，让我们看得不甚清晰：
```js
const a = 'helloworld';
export default a;
```
其实导出的是
```js
{
  "default": a
}
```
而并非 `a` 这个变量，这就是我为什么之前要强调语法糖了，如果你将 `export default` 还原为：
```js
const a = 'helloworld';
export {a as default}
```
是不是就能清楚一点了呢。

我们的疑问也就迎刃而解了。

注：以上所有代码都是在webpack开发环境（babel）中运行（保证ES6模块语法可以被识别）。

参考：[Node中没搞明白require和import，你会被坑的很惨](http://imweb.io/topic/582293894067ce9726778be9) <br>
[Javascript require vs require .default](https://stackoverflow.com/questions/43247696/javascript-require-vs-require-default)

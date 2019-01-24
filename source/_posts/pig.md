---
title: 从callback到async
date: 2019-01-24
tags: [js]
categories: js
---

本文不是概念性文章,只是通过一个例子来看看操作异步的一些手段.

## 回调函数

先看一个例子:

```js
const fs = require('fs')

fs.readFile('./file.txt', 'utf-8', (err, res) => {
  console.log(res)
})
```

通过 node 提供的 fs.readFile 方法打印出`file.txt`的内容.

如果`readFile`可以通过 promise 方式调用就好了, 像下面这样:

```js
readFile('./file.txt', 'utf-8').then(res => {
  console.log(res)
})
```

## promise

现在我们将 callback 形式的方法调用 转成 promise 形式的调用,这个过程也成为 promisify

```js
const promisify = fn => (...args) =>
  new Promise((reslove, reject) =>
    func(...args, (err, result) => (err ? reject(err) : resolve(result)))
  )
```

当然,在 node 8 及其以上版本,可以使用[util.promisify](https://nodejs.org/api/util.html#util_util_promisify_original)

现在就可以这样做啦

```js
const fs = require('fs')
const readFile = promisify(fs.readFile)
readFile('./file.txt', 'utf-8').then(res => {
  console.log(res)
})
```

统一的 API,线性的调用方式,更为灵活的[异步控制](https://zhuanlan.zhihu.com/p/29792886),exciting!

## async

ES7(ECMAScript 2016)推出了 Async 函数(async/await),实现了以顺序/同步代码的编写方式来控制异步流程.再来一遍

```js
const fs = require('fs')
const util = require('util')
const readFile = util.promisify(fs.readFile)

;(async () => {
  const res = await readFile('./package.json', 'utf-8')
  console.log(res)
})()
```

## 使用 generator 与 promise 模拟 co

相较于 promise,async/await 更易理解. 如果你听过大名鼎鼎的 co,则会发现这二者使用方式是多么相似啊!

```js
const co = require('co')
const fs = require('fs')
const util = require('util')
const readFile = util.promisify(fs.readFile)
co(function*() {
  const res = yield readFile('./package.json', 'utf-8')
  console.log(res)
})
```

不过这并不是什么黑魔法,继续向下看.

### generator

首先我们需要一点 generator 的知识,你可以看[Generator-MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)

如果你不想看,看下 demo 也可以

demo 1

```js
function* generatorFn() {
  yield 1
  yield 2
  yield 3
}

const it = generatorFn()

console.log(it.next()) // { value: 1, done: false }
console.log(it.next()) // { value: 2, done: false }
console.log(it.next()) // { value: 3, done: false }
console.log(it.next()) // { value: undefined, done: true }
```

demo 2

```js
function* generatorFn() {
  const a = yield 1
  console.log(a) // hhh
  yield 2
  yield 3
}

const it = generatorFn()
console.log(it.next()) // { value: 1, done: false }
console.log(it.next('hhh')) // { value: 2, done: false }
console.log(it.next()) // { value: 3, done: false }
console.log(it.next()) // { value: undefined, done: true }
```

弄清楚代码执行顺序即可。

#### 小练习

```js
const fs = require('fs')
const util = require('util')
const readFile = util.promisify(fs.readFile)

function go() {}

go(function*() {
  const file1 = yield readFile('./file1.txt', 'utf-8')
  console.log(file1)
  const file2 = yield readFile('./file2.txt', 'utf-8')
  console.log(file2)
})
```

#### 解答

思路很简单,我们只要在 go 中接住 yield 抛出来的 promise,注册好"解决函数"(resolver),在 resolver 内部将执行结果塞进去即可.

先写一个最挫的,只能玩 2 层,和我们的代码绑定的死死的.

```js
function go(gen) {
  const it = gen()
  const p1 = it.next().value
  p1.then(res => {
    const p2 = it.next(res).value
    p2.then(res => {
      it.next(res)
    })
  })
}
```

很明显,可以优化一波

```js
function go(gen) {
  const it = gen()

  const run = p => {
    p.then(res => {
      const { value, done } = it.next(res)
      if (done) {
        // 如果结束了就返回
        return value
      }
      run(value)
    })
  }
  const { value, done } = it.next()
  if (done) {
    return value
  }
  run(p1)
}
```

本文到此为止，代码很粗糙，下一篇文章拿 co 源码来读读。

---

推荐阅读：

- [来尝尝 Async 函数这块语法糖
  ](https://juejin.im/post/5b7ccc65f265da436631a3d9?utm_source=gold_browser_extension)
- [Promise 异步流程控制
  ](https://zhuanlan.zhihu.com/p/29792886)
- [从 for of 聊到 Generator
  ](https://juejin.im/post/5c40484bf265da61171cfb4d?utm_source=gold_browser_extension)

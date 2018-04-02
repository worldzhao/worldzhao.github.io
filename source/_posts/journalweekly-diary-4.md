---
title: 【周记】2018.03.26-2018.03.31
date: 2018-03-25
tags: [周记]
categories: 琐事
---

> 孤山寺北贾亭西，水面初平云脚低。几处早莺争暖树，谁家新燕啄春泥。乱花渐欲迷人眼，浅草才能没马蹄。最爱湖东行不足，绿杨阴里白沙堤。 　　- 白居易《钱塘湖春行》


关键字：
1. 百乐门培训
2. 订阅管理上线前优化需求
3. 回顾js异步发展史callback => promise => co + generator => async + await + promise
4. 回顾箭头函数
5. 阅读webpack官方指南/掘金小册，还有视频
6. 初读koa源码，重点：koa中间件实现原理
7. 买了15个月的腾讯云服务器
8. 明朝那些事儿【朱佑樘，朱厚照】
9. 返校

## 工作
1. 周四晚上到周六一整天都是公司新员工培训，周四晚上组队并且进行团队介绍，周五前往西溪湿地公园“寻宝”，周六培训业务相关（然而我已经回学校了）。

一点感触，不要因为自己是实习生就对别人太 过于 有礼貌，正常交流即可，可能技术比工作了五年的老哥们差了不少，但大家都会是正式员工，是平等的，不存在上下级关系，弱势久了能想象得到以后工作很麻烦。

尽量多思考，不要一有问题就发问，实在不行正常请教即可，不用摆出学生姿态，放自然点，没人在意你是实习生还是新人。

西溪湿地很漂亮，三月的尾巴离开了杭州，这个春夏过渡的很自然。

上几张图咯。

![1.jpeg](https://upload-images.jianshu.io/upload_images/4869616-4eb1aaa6ce043370.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.jpeg](https://upload-images.jianshu.io/upload_images/4869616-5b2b292a8c63aa3a.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.jpeg](https://upload-images.jianshu.io/upload_images/4869616-b7ce808e0f8b2ca2.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![4.jpeg](https://upload-images.jianshu.io/upload_images/4869616-857e88538234a72c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 订阅管理上线前优化需求。特意留了一个星期来修复bug，结果偏偏最后一天冒出四五个需求，还撞上了培训，真是全场最佳。还好有暖心的同事。

## 学习
1. 回顾js异步发展史，真的真的特别重要，原来直接跳过了generator到了async，这周静下心来梳理了一下，配合co也算是会入门使用了，但是没有去看co的源码，下周可以看看。上码：
```js
const fs = require('fs')

// 第一阶段 callback
function readFile(cb) {
  fs.readFile('./package.json', cb)
}

readFile((err, data) => {
  if (!err) {
    data = JSON.parse(data)
    console.log(data.name)
  }
})

// 第二阶段 promise
function readFileAsync(path) {
  return new Promise((reslove, reject) => {
    fs.readFile(path, (err, data) => {
      if (err) reject(err)
      else reslove(data)
    })
  })
}

readFileAsync('./package.json') 
  .then(data => {
    data = JSON.parse(data)
    console.log(data.name)
  })
  .catch(err => {
    console.log(err)
  })

// 第三阶段  co + generator function + promise
const co = require('co')
const util = require('util') 

co(function*() {
  let data = yield util.promisify(fs.readFile)('./package.json')
  data = JSON.parse(data)
  console.log(data.name)
})

// 第四阶段 async
const readAsync = util.promisify(fs.readFile)
async function readWithAsync(path) {
  let data = await readAsync(path)
  data = JSON.parse(data)
  console.log(data.name)
}

readWithAsync('./package.json')

```
2. 回顾箭头函数，上码。
```js
const luke = {
  id: 1,
  say: function() {
    setTimeout(function() {
      console.log('id:', this.id)
    }, 500)
  },
  sayWithThis: function() {
    let that = this
    setTimeout(function() {
      console.log('this id:', that.id)
    }, 1000)
  },
  sayWithArrow: function() {
    setTimeout(() => {
      console.log('arrow id:', this.id)
    }, 1500)
  },
  sayWithGlobalArrow: () => {
    setTimeout(() => {
      console.log('global arrow id:', this.id)
    }, 2000)
  }
}

luke.say()
luke.sayWithThis()
luke.sayWithArrow()
luke.sayWithGlobalArrow()
```
5. 系统性的学习了一下webpack，对webpack在浏览器上实现模块化进行了一些思考（webpack在多页应用单页应用使用情景的差别），部分笔记如下：
- ES6,7/react => babel
- css相关 => css-loader style-loader extra-text-webpack-plugin
- 代码规范 => eslint
- 图片，字体 => file-loader url-loader
- 开发热更新环境 => webpack-dev-server
- Common chunk
- Code splitting && lazy load
- Uglify && Minisize

提取公共代码：CommonChunkPlugin 通过将公共模块拆出来，最终合成的文件能够在最开始的时候加载一次，便存到缓存中供后续使用。这个带来速度上的提升，因为浏览器会迅速将公共的代码从缓存中取出来，而不是每次访问一个新页面时，再去加载一个更大的文件。4.0版本已经移除

css相关：
- css引入：
    - style-loader ：创建style标签
    - css-loader：使得css可被import
- Css modules
- 配置less/sass/stylus
- 抽离css： extra-text-webpack-plugin 可以将css或各种预处理器[转换成css]抽离出来，有利于缓存，异步组件无法抽离，需要配置fallback，不能抽离的css动态生成style标签，官方文档很完善【重复@import的css代码若不抽离会被打入js两次】

代码分割和懒加载： 不需要配置webpack，只需要自己在代码中实现即可。
- import(). [react-loadable]
- require.ensure([],function(){})
- 因为分割成了多个文件，共同依赖存在重复打包的问题，在入口处进行引入减少重复打包
- [webpack异步加载的原理](https://github.com/yongningfu/webpa_ensure)

6. koa中间件原理。koa-compose:
看源码。

好累，写不动了。

7. 看关键字
8. 看关键字
9. 看关键字
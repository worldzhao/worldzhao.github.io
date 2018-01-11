---
title: vuex-demo 记事本
date: 2017-07-08
tags: [Vue,Vuex]
categories: js
---
# vuex-demo 记事本

> A Vue.js note project for practicing vuex 
> 项目地址:[github](https://github.com/hackerwen/Vue-Note)
> 预览地址:[点击预览](http://112.74.202.2/note.html)
参考:[
实例 - Vue 单页应用：记事本](http://www.jianshu.com/p/918f1b29374d)
作者:-[MR_LP___李鹏](http://www.jianshu.com/u/5a2fd0b8fb30) 
[快速上手vuex](http://www.jianshu.com/p/04ebf09e72a1)
作者:-[PengL](http://www.jianshu.com/u/ed858918d92b) 

<!--more-->
### 预览图
![vuex-demo.gif](http://upload-images.jianshu.io/upload_images/4869616-f997ccc571163a1f.gif?imageMogr2/auto-orient/strip)


### 完成功能
1. 添加事件
2. 删除事件
3. 收藏事件
4. 本地储存

### 注意事项
1. 表单进行双向数据绑定时请务必使用v-model，不要使用双大括号的形式，不然视图会无法是刷新，没错这里说的就是textarea - -!!!
2. editingNote是通过note赋值得到的，他们指向同一块内存区域，所以对editingNote操作也是对相应的note操作,值传递和引用传递明明大一就很熟了，现在还犯错 - -!!!
3. 初始状态下的数组或对象在后面改变的长度或添加了属性，Vue无法检测到新增的项，此时应该是用Vue.set方法添加项，以便监视. 
4. vuex小结

    1. state里面就是存放的状态,我的理解是要托管共享的数据
    2. mutations就是存放如何更改状态方法
    3. getters就是从state中派生出状态，比如将state中的某个状态进行过滤然后获取新的状态(数据)。
    4. actions就是mutation的加强版，它可以通过commit mutations中的方法来改变状态，最重要的是它可以进行异步操作。
    5. modules顾名思义，就是当用这个容器来装这些状态还是显得混乱的时候，我们就可以把容器分成几块，把状态和管理规则分类来装。这和我们创建js模块是一个目的，让代码结构更清晰。
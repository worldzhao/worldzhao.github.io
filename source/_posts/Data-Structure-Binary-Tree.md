---
title: 数据结构-二叉树
date: 2017-07-18
tags: [二叉树]
categories: 数据结构
---

### 二叉树的遍历

#### 成果如图：

![动态预览.gif](http://upload-images.jianshu.io/upload_images/4869616-1aeaced9f5b5beb3.gif?imageMogr2/auto-orient/strip)

#### 思路：

1. 先对二叉树进行遍历，按照顺序将节点存入数组
2. 对数组中的节点进行样式处理（排他以及定时器）

#### 遍历代码：

1.  先序遍历：若二叉树为空，则空操作返回，否则先访问根结点，然后前序遍历左子树，再前序遍历右子树。

![先序遍历.gif](http://upload-images.jianshu.io/upload_images/4869616-3758fa00ca766609.gif?imageMogr2/auto-orient/strip)

```js
function DLR(node) {
  //先序遍历
  if (node) {
    //判断二叉树是否存在，若不存在则结束返回
    nodeArr.push(node)
    DLR(node.children[0])
    DLR(node.children[1])
  }
}
```

2.  中序遍历：若树为空，则空操作返回，否则从根结点开始（注意并不是先访问根结点），中序遍历根结点的左子树，然后是访问根结点，最后中序遍历右子树。

![中序遍历.gif](http://upload-images.jianshu.io/upload_images/4869616-084d52df07787155.gif?imageMogr2/auto-orient/strip)

```js
function LDR(node) {
  if (node) {
    LDR(node.children[0])
    nodeArr.push(node)
    LDR(node.children[1])
  }
}
```

3.  后序遍历：若树为空，则空操作返回，否则从左到右先叶子后结点的方式遍历访问左右子树，最后访问根结点。

![后序遍历.gif](http://upload-images.jianshu.io/upload_images/4869616-d2bf7e0ecada1d51.gif?imageMogr2/auto-orient/strip)

```js
function LRD(node) {
  if (node) {
    LRD(node.children[0])
    LRD(node.children[1])
    nodeArr.push(node)
  }
}
```

最后我们得到了按照遍历顺序排序的数组，再进行样式处理，代码如下

```js
function activeNode(arr) {
  for (let i = 0; i < arr.length; i++) {
    setTimeout(function() {
      for (let j = 0; j < arr.length; j++) {
        arr[j].className = ''
      } //清除所有节点样式
      arr[i].className = 'active' //给当前节点添加样式
    }, i * 500) //i*500 给定时器创造时间间隔
  }
}
```

[参考资料](https://segmentfault.com/a/1190000000740261)

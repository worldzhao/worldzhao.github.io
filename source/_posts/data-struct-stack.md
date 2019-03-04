---
title: 数据结构-栈
date: 2017-08-02
tags: [栈]
categories: 数据结构
---

初稿于 2017-08-02<br>
更新于 2019-03-04

---

## 栈结构

栈结构是动态数组的子集，我们基于 JavaScript 提供（动态）数组，通过定义相关接口 API 来实现栈这种数据结构，并且通过一些例子来加深印象。

### 定义

1.  特殊的列表，栈内的元素只能通过列表的一端访问，即栈顶

2.  后入先出(LIFO,last-in-first-out)的数据结构

### 需要实现的接口方法

1.  入栈 push

2.  出栈 pop

3.  查找 pick

4.  栈内元素总量查找 getLength

### 实现

```js
function Stack() {
  this.dataStore = []
}

// 入栈
Stack.prototype.push = function(x) {
  this.dataStore[this.dataStore.length] = x
}

// 出栈
Stack.prototype.pop = function() {
  return this.dataStore.pop()
}

// 查找
Stack.prototype.pick = function() {
  return this.dataStore[this.dataStore.length - 1]
}

Stack.prototype.getLength = function() {
  return this.dataStore.length
}

// s 就是我们的栈 我们操作对象是 s 不是 s 的 dataStore,切记

var s = new Stack()

s.push('Tencent')
s.push('Ali')
s.push('Baidu')

console.log(s.pick()) // Baidu
console.log(s.pop()) // Baidu
console.log(s.pick()) // Ali
```

### 应用

#### 回文数的判断

回文就是指一个单词，数组，短语，从前往后从后往前都是一样的 12321.abcba

回文最简单的思路就是:把元素反转后如果与原始的元素相等，那么就意味着这就是一个回文了

这里可以用到这个栈类来操作

```js
// 定义栈
var Stack = require('./stack.js')

function isPalindrome(str) {
  var s = new Stack()

  for (var i = 0; i < str.length; i++) {
    s.push(str[i])
  }

  var newStr = ''

  for (var j = 0; j < str.length; j++) {
    newStr += s.pop()
  }

  if (str === newStr) {
    return true
  } else {
    return false
  }
}

console.log(isPalindrome('112211'))
```

看看这个 isPalindrome 函数，其实就是通过调用 Stack 类，然后把传递进来的 str 这个元素给分解后的每一个组成单元给压入到栈了

根据栈的原理，后入先出的原则，通过 pop 的方法在反组装这个元素，最后比较下之前与组装后的，如果相等就是回文了

你要是用 reverse 也可以...

#### 将数字转换为二进制和八进制

```js
var Stack = require('./stack.js')

var numConvert = function(num, base) {
  var stack = new Stack()
  let converted = ''

  while (num > 0) {
    stack.push(num % base)
    num = Math.floor(num / base)
  }

  while (stack.getLength() > 0) {
    converted += stack.pop()
  }

  return converted
}

console.log(numConvert(10, 2)) // 1010
```

#### 括号匹配

```js
/*
 * @lc app=leetcode.cn id=20 lang=javascript
 *
 * [20] 有效的括号
 *
 * https://leetcode-cn.com/problems/valid-parentheses/description/
 * 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
 *
 * 有效字符串需满足：
 *
 *
 * 左括号必须用相同类型的右括号闭合。
 * 左括号必须以正确的顺序闭合。
 *
 *
 * 注意空字符串可被认为是有效字符串。
 *
 * 示例 1:
 *
 * 输入: "()"
 * 输出: true
 *
 *
 * 示例 2:
 *
 * 输入: "()[]{}"
 * 输出: true
 *
 *
 * 示例 3:
 *
 * 输入: "(]"
 * 输出: false
 *
 *
 * 示例 4:
 *
 * 输入: "([)]"
 * 输出: false
 *
 *
 * 示例 5:
 *
 * 输入: "{[]}"
 * 输出: true
 *
 */
/**
 * @param {string} s
 * @return {boolean}
 */
// const isValid = s => {
//   if (s === '') return true
//   const map = {
//     ')': '(',
//     ']': '[',
//     '}': '{'
//   }
//   const stack = []
//   for (let i = 0; i < s.length; i++) {
//     if (!stack.length) {
//       stack.push(s[i])
//     } else if (map[s[i]] === stack[stack.length - 1]) {
//       stack.pop()
//     } else {
//       stack.push(s[i])
//     }
//   }
//   return !stack.length
// }
const isValid = s => {
  if (s === '') return true
  const leftP = ['(', '[', '{']
  const map = {
    ')': '(',
    ']': '[',
    '}': '{'
  }
  const stack = []
  for (let i = 0; i < s.length; i++) {
    if (leftP.includes(s[i])) {
      stack.push(s[i])
    } else {
      const top = stack.pop()
      if (top !== map[s[i]]) {
        return false
      }
    }
  }
  return !stack.length
}
```

#### 用栈实现队列

```js
/*
 * @lc app=leetcode.cn id=232 lang=javascript
 *
 * [232] 用栈实现队列
 *
 * https://leetcode-cn.com/problems/implement-queue-using-stacks/description/
 *
 * 使用栈实现队列的下列操作：
 *
 *
 * push(x) -- 将一个元素放入队列的尾部。
 * pop() -- 从队列首部移除元素。
 * peek() -- 返回队列首部的元素。
 * empty() -- 返回队列是否为空。
 *
 *
 * 示例:
 *
 * MyQueue queue = new MyQueue();
 *
 * queue.push(1);
 * queue.push(2);
 * queue.peek();  // 返回 1
 * queue.pop();   // 返回 1
 * queue.empty(); // 返回 false
 *
 * 说明:
 *
 *
 * 你只能使用标准的栈操作 -- 也就是只有 push to top, peek/pop from top, size, 和 is empty
 * 操作是合法的。
 * 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
 * 假设所有操作都是有效的 （例如，一个空的队列不会调用 pop 或者 peek 操作）。
 *
 *
 */
/**
 * Initialize your data structure here.
 */
var MyQueue = function() {
  this.stackIn = []
  this.stackOut = []
}

/**
 * Push element x to the back of queue.
 * @param {number} x
 * @return {void}
 */
MyQueue.prototype.push = function(x) {
  this.stackIn.push(x)
  return x
}

/**
 * Removes the element from in front of queue and returns that element.
 * @return {number}
 */
MyQueue.prototype.pop = function() {
  if (!this.stackOut.length) {
    while (this.stackIn.length) {
      const top = this.stackIn.pop()
      this.stackOut.push(top)
    }
  }
  return this.stackOut.pop()
}

/**
 * Get the front element.
 * @return {number}
 */
MyQueue.prototype.peek = function() {
  if (!this.stackOut.length) {
    while (this.stackIn.length) {
      const top = this.stackIn.pop()
      this.stackOut.push(top)
    }
  }
  return this.stackOut[this.stackOut.length - 1] // 相当于栈的peek
}

/**
 * Returns whether the queue is empty.
 * @return {boolean}
 */
MyQueue.prototype.empty = function() {
  if (this.stackIn.length === 0 && this.stackOut.length === 0) {
    return true
  }
  return false
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * var obj = Object.create(MyQueue).createNew()
 * obj.push(x)
 * var param_2 = obj.pop()
 * var param_3 = obj.peek()
 * var param_4 = obj.empty()
 */
```

---
title: 从callback到async
date: 2018-11-24
tags: [策略模式]
categories: 设计模式
---

# 设计模式-策略模式

## 策略模式的定义

定义一系列的算法，把它们一个个封装起来，并使他们可以相互替换。

将可变的部分和变化的部分隔开，目的就是将算法的使用与算法的实现分离开。

> “在函数作为一等对象的语言中，策略模式是隐形的。 strategy 就是值为函数的变量。”

## 策略模式应用场景

### 优化判断语句

举一个简单的例子：

后端返回一个 status 字段，如果是 0，就代表停用，如果是 1，就代表启用，要求前端在页面上显示出相应文案。

```js
/* 解法一：三元运算 */
const text = status === 0 ? '停用' : '启用'
document.getElementById('root').innerText = 'text'

/* 解法二：if判断 */
function getText(status) {
  if (status === 0) {
    return '停用'
  } else if (status === 1) {
    return '启用'
  } else {
    return null
  }
}

document.getElementById('root').innerText = getText(status)

/* 解法三：switch语句*/

// 与if区别不大，就不做演示了
```

我们很轻松地解决了这个问题，如果 status 的可能的值不再是 0 或 1，而是 0-9，分别代表某个流程的 10 个步骤，对应 10 个不同的文案，上面的写法就有点麻烦了。

可能会变成下面的样子：

```js
function getText(status) {
  if (status === 0) {
    return '文案0'
  } else if (status === 1) {
    return '文案1'
  } else if (status === 2) {
    // ...
  }
  // ...
}

document.getElementById('root').innerText = getText(status)
```

这样编写存在的问题：

- if-else 语句需要覆盖所有逻辑分支
- 缺乏弹性，新增一种文案，则要 getText 方法内新增一句 if-else

我们要使用策略模式优化

```js
// 定义一个策略对象
const strategies = {
  0: '文案0',
  1: '文案1',
  2: '文案2',
  // ...
  9: '文案9'
}
// 通过输入[key]的不同，选择不同的策略算法[value]
const text = strategies[status]
document.getElementById('root').innerText = text
```

此处要进行的操作比较简单，根据不同的 status 返回不同普通的 text。

如果是针对 status 进行不同的操作，我们将字符串换成相应的操作方法即可。

```js
// 定义一个策略对象
const strategies = {
  0: function() {},
  1: function() {},
  2: function() {},
  2: function() {}
  // ...
}
strategies[status] && strategies[status]()
```

这里是用的是匿名函数，如果将不同条件对应的算法抽出来定义成一个普通函数，算法的复用性也得到了增强。

### 表单校验

在一个 Web 项目中，注册、登录、修改用户信息等功能的实现都离不开提交表单。

在将用户输入的数据交给后台之前，常常要做一些客户端力所能及的校验工作，比如注册的 时候需要校验是否填写了用户名，密码的长度是否符合规定，等等。这样可以避免因为提交不合 法数据而带来的不必要网络开销。

假设我们正在编写一个注册的页面，在点击注册按钮之前，有如下几条校验逻辑。

- 用户名不能为空。
- 密码长度不能少于 6 位。
- 手机号码必须符合格式。

#### 使用 if-else 进行表单校验

```html
<html lang="en">
  <head>
    <title>使用判断语句进行表单校验</title>
  </head>
  <body>
    <form action="http://xxx.com/register" id="registerForm" method="post">
      请输入用户名: <input type="text" name="userName" /> 请输入密码:
      <input type="text" name="password" />请输入手机号码:
      <input type="text" name="phoneNumber" /> <button>提交</button>
    </form>
    <script>
      var registerForm = document.getElementById('registerForm')
      registerForm.onsubmit = function() {
        if (registerForm.userName.value === '') {
          alert('用户名不能为空')
          return false
        }
        if (registerForm.password.value.length < 6) {
          alert('密码长度不能少于 6 位')
          return false
        }
        if (!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
          alert('手机号码格式不正确')
          return false
        }
      }
    </script>
  </body>
</html>
```

- registerForm.onsubmit 函数比较庞大，包含了很多 if-else 语句，这些语句需要覆盖所有的校验规则。
- registerForm.onsubmit 函数缺乏弹性，如果增加了一种新的校验规则，或者想把密码的长度校验从 6 改成 8，我们都必须深入 registerForm.onsubmit 函数的内部实现，这是违反开放—封闭原则的。
- 算法的复用性差，如果在程序中增加了另外一个表单，这个表单也需要进行一些类似的校验，那我们很可能将这些校验逻辑复制得漫天遍野

#### 使用策略模式进行优化

## 策略模式的优缺点

### 优点

- 策略模式可以有效避免多重条件选择语句
- 策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的 strategy 中，使得它
  们易于切换，易于理解，易于扩展
- 策略模式的算法可以使用在其他地方，复用性较强

### 缺点

- 需要增加策略对象

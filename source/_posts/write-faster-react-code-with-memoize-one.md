---
title: 编写更快的 React 代码（一）：memoize-one 简介
date: 2019-04-11
tags: [react, memoize-one, 性能优化]
categories: react
---

# 编写更快的 React 代码（一）：memoize-one 简介

## 引言

不同类型业务要求的性能标准各不相同。如果对一个 ToB 的后台管理系统要求首屏速度以及 SEO，显然不合理也没必要。

第一要考虑的不是如何去优化，而是值不值得去优化，React 性能已经足够优秀，毕竟“过早优化是魔鬼”，情况总是“可以，但没必要”。

作为一个开发人员，深入了解工具不足之处，并拥有对其进行优化的能力，是极其重要的。

React 性能优化大抵可分为两点：

1. 减少 rerender 次数 （immutable data、shouldComponentUpdate、PureComponent）
2. 减轻 rerender 复杂度 （memoize-one）

本文基于 memoize-one 对 render 方法进行优化，达到减轻不必要 render 复杂度的效果。

## 存在的问题

先看一个简单的组件，如下所示：

```js
class Example extends Component {
  state = {
    filterText: ""
  };

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    const filteredList = this.props.list.filter(item =>
      item.text.includes(this.state.filterText)
    );

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>
          {filteredList.map(item => (
            <li key={item.id}>{item.text}</li>
          ))}
        </ul>
      </Fragment>
    );
  }
}
```

该组件接收父组件传递的 list，筛选出包含 filterText 的 filteredList 并进行展示。

问题是什么？

在未进行任何处理的情况下，父组件 render，总会导致子组件 render，即使子组件的 state/props 并未发生变化，如果筛选的数据量大，筛选逻辑复杂，这将是一个很重要的优化点。

要达到怎样的效果？

1. state(filterText)/props(list)未发生变化时，不进行 render(引言-性能优化第 1 点)， 此处暂不讨论
2. state(filterText)/props(list)未发生变化时，进行 render，复用上一次计算结果

## memoize-one

> A memoization library which only remembers the latest invocation

### 基本使用

```js
import memoize from "memoize-one";

const add = (a, b) => a + b; // 基本计算方法
const memoizedAdd = memoize(add); // 生成可缓存的计算方法

memoizedAdd(1, 2); // 3

memoizedAdd(1, 2); // 3
// Add 函数没有被执行：上一次的结果直接返回

memoizedAdd(2, 3); // 5
// Add 函数被调用获取新的结果

memoizedAdd(2, 3); // 5
// Add 函数没有被执行：上一次的结果直接返回

memoizedAdd(1, 2); // 3
// Add 函数被调用获取新的结果
// 即使该结果在之前已经缓存过了
// 但它并不是最近一次的缓存结果，所以缓存结果丢失了
```

在了解基本使用后，我们来对上述案例进行优化。

### 优化案例

```js
import memoize from "memoize-one";

class Example extends Component {
  state = { filterText: "" };

  // 只有在list或filterText改变的时候才会重新调用真正的filter方法（memoize入参）
  filter = memoize((list, filterText) =>
    list.filter(item => item.text.includes(filterText))
  );

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    // 在上一次render后，如果参数没有发生改变，`memoize-one`会重复使用上一次的返回结果
    const filteredList = this.filter(this.props.list, this.state.filterText);

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>
          {filteredList.map(item => (
            <li key={item.id}>{item.text}</li>
          ))}
        </ul>
      </Fragment>
    );
  }
}
```

### 源码解析

如果除去 ts 相关以及注释，不到 20 行。memoize-one 本质是一个高阶函数，真正计算函数作为参数，返回一个新的函数，新的函数内部会缓存上一次入参以及上一次返回值，如果本次入参与上一次入参相等，则返回上一次返回值，否则，重新调用真正的计算函数，并缓存入参以及结果，供下一次使用。

假装这里有一张流程图 :)

```ts
// 默认比较先后入参是否相等的方法，使用者可自定义比较方法
import areInputsEqual from './are-inputs-equal';

// 函数签名
export default function<ResultFn: (...any[]) => mixed>(
  resultFn: ResultFn,
  isEqual?: EqualityFn = areInputsEqual,
): ResultFn {
  // 上一次的this
  let lastThis: mixed;
  // 上一次的参数
  let lastArgs: mixed[] = [];
  // 上一次的返回值
  let lastResult: mixed;
  // 是否已经初次调用过了
  let calledOnce: boolean = false;

  // 被返回的函数
  const result = function(...newArgs: mixed[]) {
    // 如果参数或this没有发生变化或非初次调用
    if (calledOnce && lastThis === this && isEqual(newArgs, lastArgs)) {
      // 直接返回上一次的计算结果
      return lastResult;
    }

    // 参数发生变化或者是初次调用
    lastResult = resultFn.apply(this, newArgs);
    calledOnce = true;
    // 保存当前参数
    lastThis = this;
    // 保存当前结果
    lastArgs = newArgs;
    // 返回当前结果
    return lastResult;
  };

  // 返回新的函数
  return (result: any);
}
```

#### 拓展：斐波那契数列

下面是一个计算斐波那契数列的例子，该例子使用迭代代替递归，并且利用闭包缓存之前的结果。

```js
const createFab = () => {
  const cache = [0, 1, 1];
  return n => {
    if (typeof cache[n] !== "undefined") {
      return cache[n];
    }
    for (let i = 3; i <= n; i++) {
      if (typeof cache[i] !== "undefined") continue;
      cache[i] = cache[i - 1] + cache[i - 2];
    }
    return cache[n];
  };
};

const fab = createFab();
```

## 总结

本文基于 React 介绍了 memoize-one 库的相关使用及其原理，在 React 中实现了类似与 Vue 计算属性（computed）的效果 —— 基于依赖缓存计算结果，达到减轻不必要 render 复杂度的效果。

从业务开发角度来讲，Vue 提供的 API 极大地提高了开发效率。

React 自身解决的问题并不多，但得益于活跃的社区，工作中遇到的问题总能找到解决方案，并且在摸索这些解决方案的同时，我们能够学习到诸多经典的编程思想，从而减轻对框架的依赖。

> I’ve always said that React will make you a better JavaScript developer. - Tyler McGinnis

## 参考链接

- [You Probably Don't Need Derived State](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)，by React Team
- [记忆化技术 memoize-one](https://liyang0207.github.io/2018/10/11/%E3%80%8A%E8%AE%B0%E5%BF%86%E5%8C%96%E6%8A%80%E6%9C%AFmemoize-one%E3%80%8B/)， by Leon
- [memoize-one](https://github.com/alexreardon/memoize-one)，by alexreardon
- [计算属性和侦听器](https://cn.vuejs.org/v2/guide/computed.html)，by Vue

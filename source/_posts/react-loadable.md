---
title: react-loadable原理浅析
date: 2018-02-06
tags: [react]
categories: js
---

# react-loadable

最近在学习 react，之前做的一个项目首屏加载速度很慢，便搜集了一些优化方法，`react-loadable`这个库是我在研究路由组件按需加载的过程中发现的，其实`react-router v4`官方也有`code splitting`的相关实践，但是在现在的 webpack 3.0 版本下已经不行了，因为需要使用以下相关语法

```bash
import loadSomething from 'bundle-loader?lazy!./Something'
```

有兴趣的同学可以自行研究（2018.06.14:现 react-router[code-splitting] 部分已更新为基于 react-loadable 实现，即本文实现方式）。

[react-router-v4:code-splitting](https://reacttraining.com/react-router/web/guides/code-splitting)

后面就发现了 react-loadable 这个库可以帮助我们按需加载，其实和 react-router-v4 官方实现的原理差不太多，基本使用方法如下：

第一步：先准备一个 Loding 组件,这个是官方的，你自己写一个更好：

```js
const MyLoadingComponent = ({ isLoading, error }) => {
  // Handle the loading state
  if (isLoading) {
    return <div>Loading...</div>
  }
  // Handle the error state
  else if (error) {
    return <div>Sorry, there was a problem loading the page.</div>
  } else {
    return null
  }
}
```

第二步：引入 react-loadable

```js
import Loadable from 'react-loadable'

const AsyncHome = Loadable({
  loader: () => import('../containers/Home'),
  loading: MyLoadingComponent
})
```

第三步：替换我们原本的组件

```js
<Route path="/" exact component={AsyncHome} />
```

这样，你就会发现只有路由匹配的时候，组件才被 import 进来，达到了`code splitting`的效果，也就是我们常说的按需加载， 代码分块，而不是一开始就将全部组件加载。

![chunk](http://upload-images.jianshu.io/upload_images/4869616-87139115a2c3215e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以观察到，点击不同的路由都会加载一个 chunk.js，这就是我们所分的块。

## 核心：import()

不要把 `import`关键字和`import()`方法弄混了，该方法是为了进行动态加载才被引入的。

`import`关键字的引入是静态编译且存在提升的，这就对我们按需加载产生了阻力（画外音：require 是可以动态加载的），所以才有了`import()`，而`react-loadable`便是利用了`import()`来进行动态加载。

[阮一峰：Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)

[tc39/proposal-dynamic-import](https://github.com/tc39/proposal-dynamic-import)

而且这个方法不能传变量，只能使用字符串和字符串模板，原本想将那一堆生成组件的代码进行抽象，结果死活不行，才发现坑在这里。

## Loadable

`react-loadable`有 5k star,内部机制已经十分完善了，看现在的源码我肯定看不懂，于是误打误撞地看了其`initial commit`的源码。

我们上面对`Loadable`函数的用法是这样的:

```js
const AsyncHome = Loadable({
  loader: () => import('../containers/Home'),
  loading: MyLoadingComponent
})
```

接收一个配置对象为参数,第一个属性名为`loader`，是一个方法，用于动态加载我们所需要的模块，第二个参数  就是我们的`Loading`组件咯，在动态加载还未完成的过程中会有该组件占位。

```js
{
  loader: () => import('../containers/Home'),
  loading: MyLoadingComponent
}
```

这个方法的返回值是一个 react component,我们`Route`组件和 url 相匹配时，加载的就是这个 component，该 component 通过 loader 方法进行异步加载组件以及错误处理。其实就这么多，也有点高阶组件的意思。

然后来看看源码吧(源码参数部分使用了 ts 进行类型检查)。

```js
import React from 'react'

export default function Loadable(
  loader: () => Promise<React.Component>,
  LoadingComponent: React.Component,
  ErrorComponent?: React.Component | null,
  delay?: number = 200
) {
  // 有时组件加载很快（<200ms），loading 屏只在屏幕上一闪而过。

  // 一些用户研究已证实这会导致用户花更长的时间接受内容。如果不展示任何 loading 内容，用户会接受得更快, 所以有了delay参数。

  let prevLoadedComponent = null

  return class Loadable extends React.Component {
    state = {
      isLoading: false,
      error: null,
      Component: prevLoadedComponent
    }

    componentWillMount() {
      if (!this.state.Component) {
        this.loadComponent()
      }
    }

    loadComponent() {
      // Loading占位
      this._timeoutId = setTimeout(() => {
        this._timeoutId = null
        this.setState({ isLoading: true })
      }, this.props.delay)

      // 进行加载
      loader()
        .then(Component => {
          this.clearTimeout()
          prevLoadedComponent = Component
          this.setState({
            isLoading: false,
            Component
          })
        })
        .catch(error => {
          this.clearTimeout()
          this.setState({
            isLoading: false,
            error
          })
        })
    }

    clearTimeout() {
      if (this._timeoutId) {
        clearTimeout(this._timeoutId)
      }
    }

    render() {
      let { error, isLoading, Component } = this.state

      // 根据情况渲染Loading 所需组件 以及 错误组件
      if (error && ErrorComponent) {
        return <ErrorComponent error={error} />
      } else if (isLoading) {
        return <LoadingComponent />
      } else if (Component) {
        return <Component {...this.props} />
      }
      return null
    }
  }
}
```

参考资料：

[React Loadable 介绍](http://web.jobbole.com/91704/)

[webpack v3 结合 react-router v4 做 dynamic import — 按需加载（懒加载)](https://github.com/CodeLittlePrince/blog/issues/3)

[阮一峰：Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)

[tc39/proposal-dynamic-import](https://github.com/tc39/proposal-dynamic-import)

[在 react-router4 中进行代码拆分（基于 webpack）](https://www.jianshu.com/p/547aa7b92d8c)

[react-router-v4:code-splitting](https://reacttraining.com/react-router/web/guides/code-splitting)

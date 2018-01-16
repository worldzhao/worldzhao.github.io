---
title: 【转】Build your own React Router v4
date: 2018-01-11
tags: [React Router]
categories: js
toc: true
---
# 【转】Build your own React Router v4

> 作者：Tyler <br/>
编译：胡子大哈 <br/>
翻译原文：http://huziketang.com/blog/posts/detail?postId=58d36df87413fc2e82408555 <br/>
英文原文：https://tylermcginnis.com/build-your-own-react-router-v4/

**转载请注明出处，保留原文链接以及作者信息**

---

我还记得我第一次学习开发客户端应用路由时的感觉，那时候我还是一个涉足在“单页面应用”的未出世的小伙子，那会儿，要是说它没把我的脑子弄的跟屎似的，那我是在撒谎。一开始的时候，我的感觉是我的应用程序代码和路由代码是两个独立且不同的体系，就像是两个同父异母的兄弟，互相不喜欢但是又不得不在一起。

经过了一些年的努力，我终于有幸能够教其他开发者关于路由的一些问题了。我发现，好像很多人对于这个问题的思考方式都和我当时很类似。我觉得有几个原因。首先，路由问题确实很复杂，对于那些路由库的开发者而言，找到一个合适的路由抽象概念来解释这个问题就更加复杂。第二，正是由于路由的复杂性，这些路由库的使用者倾向于只使用库就好了，而不去弄懂到底背后是什么原理。

本文中，我们会深入地来阐述这两个问题。我们会通过创建一个简单版本的 React Router v4 来解决第二个问题，而通过这个过程来阐释第一个问题。也就是说通过我们自己构建 RRv4 来解释 RRv4 是否是一个合适的路由抽象。

下面是将要用来测试我们所构建的 React Router 的代码。最终的代码实例你可以在[这里](https://codepen.io/anon/pen/MpexLL)得到。

```js

const Home = () => (
  <h2>Home</h2>
)
const About = () => (
  <h2>About</h2>
)
const Topic = ({ topicId }) => (
  <h3>{topicId}</h3>
)
const Topics = ({ match }) => {
  const items = [
    { name: 'Rendering with React', slug: 'rendering' },
    { name: 'Components', slug: 'components' },
    { name: 'Props v. State', slug: 'props-v-state' },
  ]
  return (
    <div>
      <h2>Topics</h2>
      <ul>
        {items.map(({ name, slug }) => (
          <li key={name}>
            <Link to={`${match.url}/${slug}`}>{name}</Link>
          </li>
        ))}
      </ul>
      {items.map(({ name, slug }) => (
        <Route 
          key={name} 
          path={`${match.path}/${slug}`} 
          render={() => (
            <Topic topicId={name} />
        )} />
      ))}
      <Route exact path={match.url} render={() => (
        <h3>Please select a topic.</h3>
      )}/>
    </div>
  )
}
const App = () => (
  <div>
    <ul>
      <li><Link to="/">Home</Link></li>
      <li><Link to="/about">About</Link></li>
      <li><Link to="/topics">Topics</Link></li>
    </ul>
    <hr/>
    <Route exact path="/" component={Home}/>
    <Route path="/about" component={About}/>
    <Route path="/topics" component={Topics} />
  </div>
)

```
如果你还不熟悉 React Router v4，就先了解几个基本问题。`Route` 用来渲染 UI，当一个 URL 匹配上了你所指定的路由路径，就进行渲染。`Link` 提供了一个可以浏览访问你 app 的方法。换句话讲，`Link` 组件允许你更新你的 URL，而 `Route` 组件根据你所提供的新 URL 来改变 UI。

*本文并不会手把手的教你 RRV4 的基础，所以如果上面的代码你看起来很费劲的话，可以先来这里看一下[官方文档](https://reacttraining.com/react-router/)。把玩一下里面的例子，当你觉得顺手了的时候，欢迎回来继续阅读。*

如上段所说，路由给我们提供了两个组件可以用于你的 app：`Link` 和 `Route`。我喜欢 React Router v4 的原因是它的 API “只是组件”而已，可以理解成没有引入其他概念。这就是说如果你对 React 很熟悉的话，那么你对组件以及怎么组合组件一定有自己的理解，而这对于你写路由代码依然适用。这就很方便了，因为已经熟悉了如何创造组件，那么创建你自己的 React Router 就只是做你已经熟悉的事情——创建组件。

---

worldzhao:自己从无到有搭建一个前端路由库很麻烦（比如根据url切换视图），但是依赖于React,我们可以很轻松的做到，就是render函数嘛，所以react router v4的组件和我们平时编写的组件除了功能之外并无其他区别，这便是'Just Components'的魅力了吧。

---
## Route

现在就来一起创建我们的 `Route` 组件。在上面的例子中，可以注意到 `<Route>` 使用了三个属性：`exact`、`path` 和 `component`。他们的属性类型（propTypes）对于 `Route` 组件来是这样的：

```js
static propTypes = {
  exact: PropTypes.bool,
  path: PropTypes.string,
  component: PropTypes.func,
}
```

这里有些小细节。首先，`path` 并不需要，因为如果路由中没有给 `path` 那么将会自动渲染。第二，`component` 也不需要，是因为如果路径匹配上了，有很多不同的方法来告诉 React Router 要渲染什么 UI。其中一个上面没有提到的方法就是使用 `render` 来通知 React Router，具体代码像这样：

```js
<Route path='/settings' render={({ match }) => {
  return <Settings authed={isAuthed} match={match} />
}} />
```

`render` 允许你创建一个直接返回 UI 的内联函数而不用创建额外的组件，所以我们也可以把它添加到 proTypes 中：

```js
static propTypes = {
  exact: PropTypes.bool,
  path: PropTypes.string,
  component: PropTypes.func,
  render: PropTypes.func,
}
```

现在我们知道了 `Route` 接收的属性，我们来了解一下它们的具体功能。还记得上面说的：“当 URL 匹配上了你所指定的路由 `path` 以后，`Route` 渲染其对应的 UI”。基于这样的定义，可以知道，`<Route>` 需要一些功能性函数，来判断当前的 URL 是否匹配上了组件的 `path` 属性。如果匹配上了，那么返回渲染的 UI；如果没有那么什么都不做并且返回 null。

一起来看一下这个匹配函数应该怎么写，暂且把它叫做 `matchPath` 吧。

```js
class Route extends Component {
  static propTypes = {
    exact: PropTypes.bool,
    path: PropTypes.string,
    component: PropTypes.func,
    render: PropTypes.func,
  };

  render() {
    const {path, exact, component, render} = this.props;
    const match = matchPath(window.location.pathname, {path, exact});
    if (!match) { // 如果没有匹配到，就什么都不渲染
      return null;
    }
    // 如果匹配到了 并且 component 存在
    if (component) {
      // 以 component 创建新元素并且通过 match 传递
      return React.createElement(component, {match});
    }
    // 如果匹配到了 但是没有 component ， 有 render 方法
    if (render) {
      return render({match});
    }
    return null;
  }
}
```

上面的代码即实现了：**如果匹配上了 `path` 属性，就返回 UI，否则什么也不做。**

我们再来谈一下路由的问题。在客户端应用这边，一般来讲只有两种方式更新 URL。

1. 一种是用户点击 a 标签。

2. 一种是点击后退/前进按钮。

基本上我们的路由只要关心 URL 的变化并且返回相应的 UI 即可。

---

worldzhao: 就是我们的Route组件要根据url的变化来update，监听url变化重新渲染Route组件(forceupdate).

---

假设我们知道更新 URL 的方式只有上面两种，那么就可以针对这两种情况做特殊处理了。稍后在构建 <Link> 组件的时候再详细介绍 a 标签的情况，这里先讨论后退/前进按钮。 

React Router 使用了 History工程里的 .listen 方法来监听当前 URL 的变化，为了避免再引入其他的库，我们使用 HTML5 的 popstate 事件来实现这一功能。

当用户点击了后退/前进按钮，popstate 就被触发，我们需要的就是这个功能。

因为 Route 渲染 UI 是根据当前 URL来做的，因此给 Route 配上监听能力也是合理的，在 popstate 触发的地方重新渲染 UI。就是说在触发 popstate 时检查是否匹配上了新的 URL，如果是则渲染 UI，如果不是，什么也不做，下面看一下代码。

```js
class Route extends Component {
  static propTypes: {
    path: PropTypes.string,
    exact: PropTypes.bool,
    component: PropTypes.func,
    render: PropTypes.func,
  }
  componentWillMount() {
    // 监听前进后退事件
    addEventListener("popstate", this.handlePop)
  }
  componentWillUnmount() {
    removeEventListener("popstate", this.handlePop)
  }
  handlePop = () => {
    // 重新刷新
    this.forceUpdate()
  }
  render() {
    const {
      path,
      exact,
      component,
      render,
    } = this.props
    const match = matchPath(location.pathname, { path, exact })
    if (!match)
      return null
    if (component)
      return React.createElement(component, { match })
    if (render)
      return render({ match })
    return null
  }
}
```

这里要注意的是我们只是加了一个 `popstate` 监听，当 `popstate` 触发的时候，调用 `forceUpdate` 来强制做重新渲染的判断。

这样就实现了所有的 `<Route>` 都会监听，根据后退/前进按钮来“重匹配”、“重判断”和“重渲染”。

到现在，我们一直还没有实现的是 matchPath 函数。这个函数在我们的 router 中是特别关键的，因为它是判断当前 URL 是否匹配上了 <Route> 组件的关键点。matchPath 值得注意的一点是一定要把 <Route> 的 exact 考虑清楚。如果你对 exact 还不了解，看下下面这句话，给出了规范文档中的解释：

> 只有当所给路径精确匹配上 location.pathname 时才返回 true。

![exact](https://huzidaha.github.io/images-store/201703/16-2.png)

接下来就来具体实现 `matchPath` 函数。如果你回头看一下上面 `Route` 组件的代码，你可以看到 `matchPath` 函数是这样的：

```js
const match = matchPath(window.location.pathname, {path, exact});
```

这里的match要么是对象，要么是null，这得取决于是否匹配上path。根据这个声明，我们来写matchPath代码：

```js
const matchPath = (pathname, options) => {
  const {exact = false, path} = options;
}
```

这里使用 ES6 语法。上面的意思是，创建一个叫做 exact 的变量，使其等于 options.exact，并且如果非 null 的话则设置其为 false。同样创建一个叫做 path 的变量，使其等于 options.path。

接下来就添加判断是否匹配。React Router 使用 [pathToRegex](https://github.com/pillarjs/path-to-regexp) 来实现，只需要写简单的正则匹配就可以了。

```js
const matchPath = (pathname, options) => {
  const {exact = false, path} = options;
  if (!path) {
    return {
      path: null,
      url: pathname,
      isExact: true,
    }
  }
  const match = new RegExp(`^${path}`).exec(pathname);
}
```

如果匹配上了，那么返回一个包含有所有匹配串的数组，否则返回 null。

下面是我们示例 app 的路由 '/topics/components' 的一些匹配项。

![return value](https://huzidaha.github.io/images-store/201703/16-3.png)

> 注意：每个 `<Route>` 都在自己的渲染方法里调用 `matchPath`，所以要为每个 `<Route>` 配一个 `match`。

现在我们要做的是添加判断是否有匹配的代码：

```js
const matchPath = (pathname, options) => {
  const {exact = false, path} = options;
  if (!path) {
    return {
      path: null,
      url: pathname,
      isExact: true,
    }
  }
  const match = new RegExp(`^${path}`).exec(pathname);
  if (!match) {
    // 没有匹配上
    return null;
  }
  const url = match[0];
  const isExact = pathname === url;
  if (exact && !isExact) {
    // 要求精确匹配（exact为真） 却不是精确匹配(isExact为假)
    return null;
  }
  return {
    // 不要求精确匹配(exact === false) 或是精确匹配成功(exact === isExact === true)
    path,
    url,
    isExact
  }
};
```

---
## Link

之前有讲过的，对于用户来讲，有两种方式更新 URL：通过后退/前进按钮和通过点击 a 标签。对于后退/前进点击来说，使用 `popstate` 事件给 `Route` 添加监听就可以，现在来看一下如何通过 `Link` 解决 a 标签问题。

`Link` 的 API 如下：

```js
<Link to='/some-path' replace={false} />
```

这里 `to` 是一个 string 类型，指的是要链接到的地址。`replace` 是一个布尔值，如果是 true，那么点击链接将替换当前的实体到历史堆栈，而不是添加一个新的进去。

添加这些 propTypes 到 `Link` 组件就得到：

```js
class Link extends Component {
  static propTypes = {
    to: PropTypes.string.isRequired,
    replace: PropTypes.bool,
  }
}

```

我们知道在 `Link` 组件中的渲染函数需要返回一个 a 标签，但是我们不想每次变路由都进行一次全页面刷新，所以通过增加一个 `onClick` 处理程序来劫持 a 标签。

```js

class Link extends Component {
  static propTypes = {
    to: PropTypes.string.isRequired,
    replace: PropTypes.bool,
  }
  handleClick = (event) => {
    const { replace, to } = this.props
    event.preventDefault()
    // 这里是路由,仅仅改变url（不会刷新页面）
  }
  render() {
    const { to, children} = this.props
    return (
      <a href={to} onClick={this.handleClick}>
        {children}
      </a>
    )
  }
}

```

ok，代码写到现在，就差更改当前 URL 了。在 React Router 是使用 [History 工程](https://github.com/reacttraining/history)里面的 `push` 和 `replace` 方法。为了避免增加新依赖，这里我使用 HTML5 的 `pushState` 和 `replaceState`。

>本文中我们为了防止引入额外的依赖，一直也没采用 History 库。但是它对真实的 React Router 却是至关重要的，因为它对不同的 session 管理和不同的浏览器环境进行了规范化处理。

`pushState` 和 `replaceState` 都接收三个参数。第一个参数是一个与历史实体相关联的对象，我们不需要，所以设置成一个空对象。第二个参数是标题，我们也不需要，所以也设置成空。第三个是我们需要使用的，指的是：相关 URL。

```js
const historyPush = (path) => {
  history.pushState({}, null, path)
}
const historyReplace = (path) => {
  history.replaceState({}, null, path)
}

```

在 `Link` 组件内部，会调用 `historyPush` 或者 `historyReplace`，依赖于前面提到的 `replace` 属性。

```js
class Link extends Component {
  static propTypes = {
    to: PropTypes.string.isRequired,
    replace: PropTypes.bool,
  }
  handleClick = (event) => {
    const { replace, to } = this.props
    event.preventDefault()
    replace ? historyReplace(to) : historyPush(to)
  }
  render() {
    const { to, children} = this.props
    return (
      <a href={to} onClick={this.handleClick}>
        {children}
      </a>
    )
  }
}
```

---

## Link -> Url -> Route 

现在就只剩下最后一件很关键的问题了，如果你想把上面的例子用在自己的路由代码里面，你需要注意这个问题。

当你浏览时，URL 会发生改变，但是 UI 却没有刷新，这是为什么呢？

这是因为，尽管你通过 `historyReplace` 或者 `historyPush` 改变了地址，但是 `<Route>` 并没有意识到已经改变了，也不知道应该重匹配和重渲染。为了解决这个问题，需要跟踪每一条 `<Route>` 并且当路由发生改变的时候调用 `forceUpdate`。

> React Router 通过设置状态、上下文和历史信息的组合来解决这个问题。监听路由组件的内部代码。

为了使路由简单，我们通过把所有路由对象放到一个数组里的方式来实现 `<Route>` 跟踪。每当发生地址改变的时候，就遍历一遍数组，调用相应对象的 `forceUpdate` 函数。

```js
let instances = []
const register = (comp) => instances.push(comp)
const unregister = (comp) => instances.splice(instances.indexOf(comp), 1)
```

注意这里创建了两个函数。当 `<Route>` “装配”上，就调用 `register`；当“解装配”，就调用 `unregister`。然后只要调用 `historyPush` 或者 `historyReplace`（实际上用户每次点击 <Link> 都会调用），就遍历对象数组，并调用 `forceUpdate`。

首先更新 `<Route>` 组件：

```js
class Route extends Component {
  static propTypes: {
    path: PropTypes.string,
    exact: PropTypes.bool,
    component: PropTypes.func,
    render: PropTypes.func,
  }
  componentWillMount() {
    addEventListener("popstate", this.handlePop)
    // 丢进公共数组instance中
    register(this)
  }
  componentWillUnmount() {
    unregister(this)
    removeEventListener("popstate", this.handlePop)
  }
...
}
```

再更新 `historyPush` 和 `historyReplace`：

```js
const historyPush = (path) => {
  history.pushState({}, null, path)
  // url变化后 遍历所有Route，刷新
  instances.forEach(instance => instance.forceUpdate())
}
const historyReplace = (path) => {
  history.replaceState({}, null, path)
  instances.forEach(instance => instance.forceUpdate())
}
```

这时只要 `<Link>` 被点击并且地址发生变化，每个 `<Route>` 都会接收到消息，并且进行重匹配和重渲染。

这就完成了所有的路由代码了，并且实例 app 用这些代码可以完美运行！

```js
import React, {Component} from 'react';
import PropTypes from 'prop-types';

let instances = [];
const register = (comp) => instances.push(comp);
const unregister = (comp) => instances.splice(instances.indexOf(comp), 1);

// Route
const matchPath = (pathname, options) => {
  const {exact = false, path} = options;
  if (!path) {
    return {
      path: null,
      url: pathname,
      isExact: true,
    }
  }
  const match = new RegExp(`^${path}`).exec(pathname);
  if (!match) {
    // 没有匹配上
    return null;
  }
  const url = match[0];
  const isExact = pathname === url;
  if (exact && !isExact) {
    // 要求精确匹配（exact为真） 却不是精确匹配(isExact为假)
    return null;
  }
  return {
    // 不要求精确匹配(exact === false) 或是精确匹配成功(exact === isExact === true)
    path,
    url,
    isExact
  }
};
export class Route extends Component {
  static propTypes = {
    exact: PropTypes.bool,
    path: PropTypes.string,
    component: PropTypes.func,
    render: PropTypes.func,
  };

  componentWillMount() {
    // 监听前进后退事件
    window.addEventListener("popstate", this.handlePop)
    register(this);
  }

  componentWillUnmount() {
    window.removeEventListener("popstate", this.handlePop)
    unregister(this);
  }

  handlePop = () => {
    this.forceUpdate()
  };

  render() {
    const {path, exact, component, render} = this.props;
    const match = matchPath(window.location.pathname, {path, exact});
    if (!match) { // 如果没有匹配到，就什么都不渲染
      return null;
    }
    // 如果匹配到了 并且 component 存在
    if (component) {
      // 以 component 创建新元素并且通过 match 传递
      return React.createElement(component, {match});
    }
    // 如果匹配到了 但是没有 component ， 有 render 方法
    if (render) {
      return render({match});
    }
    return null;
  }
}

// Link
const historyPush = (path) => {
  window.history.pushState({}, null, path);
  instances.forEach(instance => instance.forceUpdate());
};

const historyReplace = (path) => {
  window.history.replaceState({}, null, path);
  instances.forEach(instance => instance.forceUpdate());
};

export class Link extends Component {
  static propTypes = {
    to: PropTypes.string.isRequired,
    replace: PropTypes.bool,
  };

  handleClick = (event) => {
    const {replace, to} = this.props;
    event.preventDefault();
    // 这里是路由,仅仅改变url（不会刷新页面）
    replace ? historyReplace(to) : historyPush(to);
  };

  render() {
    const {to, children} = this.props;
    return (
      <a href={to} onClick={this.handleClick}>
      {children}
      </a>
    )
  }
}

```

另外：React Router API 还自然派生出了 `<Redirect>` 组件。使用上面我们写的代码，这个组件可以直接写成：

```js
class Redirect extends Component {
  static defaultProps = {
    push: false
  }
  static propTypes = {
    to: PropTypes.string.isRequired,
    push: PropTypes.bool.isRequired,
  }
  componentDidMount() {
    const { to, push } = this.props
    push ? historyPush(to) : historyReplace(to)
  }
  render() {
    return null
  }
}
```

注意这个组件并不渲染任何 UI，它只用来做路由定向使用。

我希望这篇文章已经帮助你搭建出了一个React Router底层模型，并且能够体会到React Router的优雅以及‘仅仅是组件’的理念。我之前总是说React会使你成为一个更好的JavaScript开发者。我现在依旧相信React Router会是你成为一个更好的React开发者。因为一切皆组件，如果你会React，你就会React Router。

I hope this has helped you create a better mental model of what’s happening in React Router while also helping you to gain an appreciation for React Router’s elegance and “Just Components” API. I’ve always said that React will make you a better JavaScript developer. I now also believe that React Router will make you a better React developer. Because everything is just components, if you know React, you know React Router.
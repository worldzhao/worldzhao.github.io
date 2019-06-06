---
title: 编写干净的代码
date: 2019-05-15
tags: [js, react]
categories: react
---

## 项目基础配置

工欲善其事必先利其器，一个好的项目架子（目录结构以及代码风格，即规范）就是一套教程。

规范是为了使得团队成员**协作**更为轻松而存在的，应当在个人习惯以及团队协作之间取得平衡，有红线、也有金线。

统一的规范也是使得团队发挥出 `1 + 1 > 2` 能力的基础。

必须遵循两个基本原则：

1. 少数服从多数（协商讨论）；
2. 用工具统一风格（lint 工具）。

常用工具：

- eslint
- editorconfig
- husky[pre-commit]
- vscode plugins
- 贯彻执行

> [推荐阅读-如何保障前端项目的代码质量-掘金](https://juejin.im/entry/5b911ff9e51d450e65482870)

## 代码相关

> 关键字：简单、可读、好维护

一切为了简单、可读，好维护。

### 策略模式

使用策略模式优化 if-else，换汤(switch)不换药(if)，策略(map)更有效

```js
// bad
const { channel } = this.state;
let chal = '';
switch (channel) {
  case 'weixin':
    chal = 1;
    break;
  case 'weixinGroup':
    chal = 2;
    break;
  case 'qq':
    chal = 3;
    break;
  case 'qrCode':
    chal = 4;
    break;
}
```

```js
// good
const { channel } = this.state;
const channelCodeMap = {
  weixin: 1,
  weixinGroup: 2,
  qq: 3,
  chal: 4,
};
const chal = channelCodeMap[channel] || '';
```

此处是根据 channel 拿取对应 code，如果是根据后端 code 显示文案同理，如果是根据后端 code 进行不同操作也是同理，只不过将键值对的值更换为我们抽象的方法而已，将不同策略抽取出去，策略对象只负责选取策略，分离可变与不可变，即为策略模式。

想要深入戳此处：

> [推荐阅读-JavaScript 复杂判断的更优雅写法-掘金](https://juejin.im/post/5bdfef86e51d453bf8051bf8) > [推荐阅读-设计模式-策略模式-我的博客](https://worldzhao.github.io/2018/11/23/design-pattern-strategy/)

### 定义常量

倘若一个字符串或是一个数字在项目中反复被使用（单个文件除外），就应当抽离出配置文件，而非在业务代码中写满意义不明的 number 与 string

```js
// bad
const { channel } = this.state;
const channelCodeMap = {
  weixin: 1,
  weixinGroup: 2,
  qq: 3,
  chal: 4,
};
const chal = channelCodeMap[channel] || '';
```

```js
// good
// constants.js
export const CHANNEL_CODES = {
  WEIXIN: 1,
  WEIXIN_GROUP: 2,
  QQ: 3,
  CHAL: 4,
};

// index.js
import { CHANNEL_CODES } from 'path2config/constants';
const { channel } = this.state;
const channelCodeMap = {
  weixin: CHANNEL_CODES.WEIXIN,
  weixinGroup: CHANNEL_CODES.WEIXIN_GROUP,
  qq: CHANNEL_CODES.WEIXIN_QQ,
  chal: CHANNEL_CODES.WEIXIN_CHAL,
};
const chal = channelCodeMap[channel] || '';
```

### 使用 async/await

除开极其复杂的异步流程（极少情况，我没碰到过）需要使用 Promise，绝大多数情况 async/await 就可以 hold 住，并且少了括号嵌套，可读性极佳。

```js
// bad
/* 获取用户信息后获取用户提现信息 */
getWithdrawInfo = () => {
  getUserInfo()
    .then(res => {
      return getWithdrawInfo(res.userId);
    })
    .then(withdrawInfo => {
      this.setState({
        withdrawInfo,
      });
    })
    .catch(err => {
      console.log(err);
    });
};
```

```js
// good
/* 获取用户信息后获取用户提现信息 */
getWithdrawInfo = async () => {
  const userInfo = await getUserInfo();
  const withdrawInfo = await getWithdrawInfo(userInfo.userId);
  this.setState({
    withdrawInfo,
  });
};
```

> [推荐阅读-从 callback 到 async-我的博客](https://worldzhao.github.io/2019/01/23/pig/)

### 单一职责

一个方法做一件事，生命周期函数力求清晰简单，内部不要有业务逻辑，简单调用业务方法即可。

bad patter 就是在生命周期内写一大堆业务逻辑，看得头皮发麻，此处不做示例。

```js
// good
componentDidMount() {
    // 获取账户信息
    this.getAccountInfo();
    // 获取提现配置信息
    this.getWithdrawConfig();
    // 获取收听时长
    this.getListenData();
}
```

### 类名拼接

使用 classnames 库（声明式）拼接复杂类名，而非各种拼接字符串（过程式）。

```js
// bad
// 这个例子写的太累了。。。
const Button = props => {
  const { size, type, openPrefixCl, prefixCl } = props;
  let btnCls = 'test-btn';
  if (size === 'large') {
    btnCls += ' test-btn__large';
  } else if (size === 'medium') {
    btnCls += ' test-btn__medium';
  } else {
    btnCls += ' test-btn__small';
  }

  if (type === 'primary') {
    btnCls += ' test-btn__primary';
  } else if (type === 'warning') {
    btnCls += ' test-btn__warning';
  } else if (type === 'danger') {
    btnCls += ' test-btn__danger';
  } else {
    btnCls += ' test-btn__default';
  }

  if (openPrefixCl === 1) {
    // 这个例子放在这里不合理 只是为了说明classnames的好处
    btnCls += ` ${prefixCl}`;
  }
  return <button className={btnCls}>点我</button>;
};
```

```js
// good
import cx from 'classnames';

const sizeClsMap = {
  small: 'test-btn__large',
  medium: 'test-btn__medium',
  large: 'test-btn__small',
};

const typeClsMap = {
  primary: 'test-btn__primary',
  warning: 'test-btn__warning',
  danger: 'test-btn__danger',
};

const Button = props => {
  const { size, type, openPrefixCl, prefixCl } = props;
  const sizeCl = sizeClsMap[size] || 'test-btn__medium';
  const typeCl = typeClsMap[type] || 'test-btn__default';
  const btnCls = cx({
    'test-btn': true,
    [typeCl]: true,
    [sizeCl]: true,
    [prefixCl]: openPrefixCl === 1,
  });
  return <button className={btnCls}>点我</button>;
};
```

### （伪）计算属性

通过 get 关键字，抽离计算逻辑，根据 state 与 props 计算出衍生值。

```js
// 使用 getters 封装 render 所需要的状态或条件的组合
// 对于返回 boolean 的 getter 使用 is- 前缀命名

// bad
render() {
    const { age } = this.state;
    const { school } = this.props;
    return (
        <>
        {
           age > 18 && (school === 'A' || school === 'B')
            ? <VipComponent />
            : <NormalComponent />
        }
        </>
    )
}
```

```js
// good
get isVIP() {
    const { age } = this.state;
    const { school } = this.props;
    return age > 18 && (school === 'A' || school === 'B')
    }

render() {
    return (
        <>
        {this.isVIP ? <VipComponent /> : <NormalComponent />}
        </>
    )
}
```

为什么是（伪），因为通过这种效果模拟出来的计算属性和 Vue 提供的计算属性有本质区别。

对于 Vue 计算属性，Vue 官方文档中存在解释：”我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是计算属性是基于它们的依赖进行缓存的。只在相关依赖发生改变时它们才会重新求值。”

假使存在两个 state 属性 A.B，计算属性只依赖 A，不依赖 B。倘若 B 变化，计算属性不会重新计算。

但在 React 中，依旧会执行 render，所以 get 没有缓存，只是个语法糖。

> [推荐阅读-使用 get 关键字优雅处理表单联动-我的博客](https://worldzhao.github.io/2019/03/16/react-computed/)

### 降低 jsx 复杂度

render(jsx) 方法复杂到一定程度时，需要合理拆分成不同子组件或子方法（单一职责）。

bad pattern 不作展示，想像一个 200 行的 render 方法并且 jsx 中充斥着各种判断判断逻辑即可。

以一个模态框组件为例，模态框 Header 以及 Footer 基本都是可配置的，会存在许多判断，将其抽离分而治之可以更干净。

只要合理抽象方法，代码基本不会有太大问题。

```js
renderModalHeader = () => {};

renderModalContent = () => {};

renderModalFooter = () => {};

render() {
    return (
      <div className="test-modal--container">
        {this.renderModalHeader()}
        {this.renderModalContent()}
        {this.renderModalFooter()}
      </div>
    );
  }
```

### 其他

- if 判断语句不通过尽早返回，避免一大堆逻辑后来个 else，很多时候直接 return 更清晰

- 在合适的场景使用 reduce/map/filter/some/every（声明式） 等方法替换 for 循环（过程式）
- 配置 webpack alias 避免导入模块相对路径过长的问题

```js
// bad
import { Modal } from '../../../components/';
```

```js
// good
import { Modal } from '@components';
```

- 公共组件需要有 propTypes 以及 defaultPropTypes 静态属性

看了 propTypes 属性就基本了解了组件的使用方式，代码即文档，如果 typescript 更好（interface）

- 相同的组件功能使用 HOC / render props/hooks 优化（高阶组件需要注意属性覆盖与静态方法丢失）

---
title: 使用eslint和editorconfig规范代码
date: 2018-01-17
tags: [eslint]
categories: 环境配置
---
# 使用eslint和editorconfig规范代码

> 该文章总结自：[慕课网: Webpack+React全栈工程架构项目实战精讲](https://coding.imooc.com/class/161.html) <br>
参考：<br>[vue项目配置eslint(附visio studio code配置)](https://www.jianshu.com/p/c94db34e525b) <br>
[VS Code里面怎么根据eslint来格式化代码？-刘祺的回答](https://www.zhihu.com/question/52777843)<br>
[VS Code中的插件以及相关配置](http://idealife.github.io/2017/06/29/VS-Code%E4%B8%AD%E7%9A%84%E6%8F%92%E4%BB%B6%E4%BB%A5%E5%8F%8A%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE/)

## 为什么要使用这些

1. 规范代码有利于团队协作
2. 纯手工规范费时费力且不能保证准确性
3. 能配合编辑器自动提醒错误，提高开发效率

## eslint

### eslint是什么

elint是随着ECMAScript版本一直更新的Js lint工具，插件丰富，并且能够套用规范，规则非常丰富，能够满足大部分团队的需求。

### eslint的使用

0. 全局安装eslint及其依赖

```bash
npm i eslint -g
```

```bash
npm i babel-eslint \
eslint-config-airbnb \
eslint-loader \
eslint-plugin-import \
eslint-plugin-jsx-a11y \
eslint-plugin-node \
eslint-plugin-promise \
eslint-plugin-react -g
```

1. 在项目中安装eslint及其依赖
```bash
npm i eslint -D
```

```bash
npm i babel-eslint \
eslint-config-airbnb \
eslint-loader \
eslint-plugin-import \
eslint-plugin-jsx-a11y \
eslint-plugin-node \
eslint-plugin-promise \
eslint-plugin-react -D
```
2. 在项目根目录下新建 `.eslintrc` 文件，如下图所示：

![.eslintrc文件](http://upload-images.jianshu.io/upload_images/4869616-ea50ea1796d329f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 编写规则
```json
{
  "parser": "babel-eslint", // 使用babel-eslint而非eslint自带的parser
  "env": {
    "browser": true,        // 代码执行环境： 浏览器 可以使用window的环境变量
    "es6": true,
    "node": true            // 可以使用node的一些环境变量
  },
  "parserOptions": {
    "ecmaVersion": 6,       // ecma语言版本
    "sourceType": "module"
  },
  "extends": "airbnb",      // 继承airbnb规范
  "rules": {
    "semi": [0],             // 不检测分号 0 = off, 1 = warn, 2 = error
    "react/jsx-filename-extension": [0] // 允许在js文件中编写jsx
  }
}
```
4. 配置webpack
```js
{
  enforce: 'pre',           // 在webpack编译之前进行检测
  test: /.(js|jsx)$/,
  loader: 'eslint-loader',
  exclude: [                // 除去node_modules
    path.resolve(__dirname, '../node_modules')
  ]
},
```

5. 启动webpack,你就会发现一大堆报错信息

![报错信息](http://upload-images.jianshu.io/upload_images/4869616-8ab52722d03f6058.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个一个找吧，考验耐心的时刻，原来我脾气这么好 :P

如果不知道错误是什么意思，看见报错后面的灰色字体吗，那个就是规则名，copy一下去[eslint rule](https://eslint.org/docs/rules/)查看规则详情。

### 配置控制台的eslint检测环境

如果你想在控制台查看项目中所有的warning 和 error，即手动在控制台通过eslint命令进行代码检测，就需要保证依赖包都在全局环境目录下安装。

然后执行
```bash
eslint --ext .js --ext .jsx client/
```
命令即可。

 --ext用来指定检测的文件格式，client/是检测的目录。

 也可以给该指令提供一个更好记的别名，编辑package.json，在script属性中新增一条。即可通过 npm run lint来检测项目中的warning 和 error了。

```bash
"scripts": {
   "lint": "eslint --ext .js --ext .jsx client/"
 },
```

如何给项目配置eslint到这里就讲完了，最后说下问什么要在全局环境下安装依赖包吧。

只有全局环境下安装了eslint才能执行 eslint --init 和 eslint --ext .js,.vue src 等eslint指令。
当项目执行eslint检测时，会先检测全局环境下有没有eslint，显然文章第一步就是在全局安装了，那么全局环境下的eslint引用依赖包时也只会在全局环境下查找。
如果你现在或之后不需要给项目初始化一个eslint配置，也不需要在控制台输出所有的warning和error，那么就不要全局环境下的eslint。执行 npm configs 查看全局环境下的包的安装路径，如果发现有eslint就删掉好了。
[这一段摘自[简书](https://www.jianshu.com/p/c94db34e525b)]

上面文章中评论有一位朋友说道：“eslint不一定需要全局安装
本地安装的话可以通过./node_modules/.bin/eslint --init来运行”

不愿意全局安装诸多依赖包的同学可以一试。

### eslint配合git

为了最大程度控制每个人的规范，我们可以在git commit代码的时候，使用git hook调用eslint进行代码规范验证，不规范的代码无法提交到仓库。

1. 安装husky (哈士奇:D)
```bash
npm i husky -D
```

2. 修改package.json的scripts
```json
"scripts": {
  ...
  "lint": "eslint --ext .js --ext .jsx client/",
  "precommit": "npm run lint"
}
```
precommit是一个钩子，当执行git commit的时候，只有通过了precommit才能够执行成功（注意，此时的eslint是在控制台通过全局命令`eslint`运行的,所以之前需要全局安装eslint以及依赖插件（第0步)。


## editorconfig

不同编辑器对文本的格式会有一定的区别，如果不统一一些规范，可能你跟别人合作的时候，每次更新下来别人的代码就会一大堆报错。

诸如：indent_style/indet_size/charset/end_of_line/insert_final_newline/trim_trailing_whitespace等等差异都会造成问题。

0. 安装插件(webstorm跳过) 此处以VS Code为例

    去商店中搜索 EditorConfig for VS Code进行安装，这样VS Code就会优先根据项目根目录的.editorconfig文件对缩进之类风格进行配置，覆盖VS Code默认配置。

    EditorConfig 插件会从文件所在目录开始逐级向上查找 .editorconfig，直到到达根目录或者找到包含属性 root=true 的 .editorconfig 文件为止。

    当找到所有满足条件的 .editorconfig 配置文件后，插件会读取这些配置文件的内容，距离文件路径最短的配置文件中的属性优先级最高，同一个文件按照从上到下方式读取，底部定义的同名属性优先级要高于顶部定义的。

    大部分编辑器都有这个插件，即使团队成员使用不同的IDE，也可以很好的统一代码风格。

    只要保证.editorconfig这个文件一直即可。

1. 在项目根目录新建 `.editorconfig`

![editorconfig](http://upload-images.jianshu.io/upload_images/4869616-8a9693e08bdd7669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 编写配置文件
```
root = true                     // 表示当前是项目根目录

[*]                             // 所有文件都使用配置
charset = utf-8                 // 编码格式
indent_style = space            // Tab键缩进的样式，由space和table两种 一般代码中是space
indent_size = 2                 // 缩进size为2
end_of_line = lf                // 以lf换行 默认win为crlf mac和linux为lf
insert_final_newline = true     // 末尾加一行空行
trim_trailing_whitespace = true // 去掉行尾多余空格
```

注： 如果发现IDE自动的格式化与eslint规则造成了冲突，记得自己去更改格式化规则。

在VS Code中，点击 文件>首选项>设置

在搜索框中输入“eslint.autoFixOnSave”

别忘了先在扩展商店安装好"ESLint"这个插件哦。

这样的话无需自己手动格式化，保存的时候便格式化成功了，如果出现错误还会出现波浪线友好提示。

小tip：如果有一些规则我不想设置完全失效，但是的确有一行代码我不能让他检查怎么办？

答： 在该代码后面添加注释
```js
import App from './App.jsx'; // eslint-disable-lines
```
更多学习请参考官方文档：
1. ESlint 官方网站：https://eslint.org/
2. editorconfig 官方网站: http://editorconfig.org/

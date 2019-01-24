---
title: babel学习笔记
date: 2018-10-13
tags: [webpack,babel]
categories: 工程化
---

## [babel](https://babeljs.io/)

```
原始代码 --> [Babel Plugin] --> 转换后的代码
```

### 转译

安装以下依赖:

1.  `@babel/core`调用 Babel 的 API 进行转码
2.  `babel-loader`[webpack 的 loader]（https://github.com/babel/babel-loader)
3.  `@babel/preset-env` 在没有任何配置选项的情况下, babel-preset-env 与 babel-preset-latest（或者 babel-preset-es2015, babel-preset-es2016 和 babel-preset-es2017 一起）的行为完全相同.
4.  ~~`@babel/preset-stage-0`稻草人提案(babel7 已移除)~~通过 npx babel-upgrade --write 升级为各种插件
5.  `@babel/preset-react`react 语法转译
    preset 即 plugin 的套餐, 无需针对一个个语法规则去安装 plugin

注:[plugin 分为转译 plugin/语法 plugin/混合 plugin](https://babel.docschina.org/docs/en/6.26.3/plugins#es2015)

```
env = es2015 + es2016 + es2017
stage-0 = stage-1 + stage-2 + stage-3
```

preset 具体包含了哪些插件可去官网查询.

通过配置 targets 可以避免浏览器已经支持的特性被转译, 如果同时设置"IE >= 9"与 "chrome >= 66", 以 "IE >= 9" 为准进行转译

### polyfill

| 方案                                                    | 优点                                                    | 缺点                                   | 推荐使用环境       |
| ------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------- | ------------------ |
| @babel/runtime + @babel/plugin-transform-runtime        | 按需引入, 打包体积小, 移除冗余工具函数(helper function) | 不模拟实例方法                         | 开发库、工具中使用 |
| @babel/polyfill                                         | 完整模拟 ES2015+环境                                    | 体积过大, 污染全局对象和内置的对象原型 | 应用中使用         |
| @babel/preset-env[useBuiltIns:"entry"] + babel-polyfill | 按~~需~~浏览器引入, 以 target 最低要求为准              | 可配置性高                             | -                  |

注:方案 1 中的@babel/runtime + @babel/plugin-transform-runtime 在 babel7 下只包含 helper function(即 Babel 进行处理时需要的帮助函数), 如果想实现 polyfill , 需要使用@babel/runtime-corejs2, [升级详情](https://babel.docschina.org/docs/en/v7-migration#babel-runtime-babel-plugin-transform-runtime)

```
["@babel/plugin-transform-runtime",  {
    "corejs": 2
}],
```

要实现真正的按需引入,即使用什么新特性打包什么新特性,可以使用实验性的 useBuildIns:"usage".

### [Babel 正在向每个文件中注入 helper 并使代码膨胀](https://github.com/babel/babel-loader)

解决方案:[@babel/runtime + @babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)

```
{
  "plugins":["@babel/plugin-transform-runtime", {
            "corejs": false,
            "helpers": true,
            "regenerator": false,
            "useESModules": false
        }],
}
```

### 在 webpack 中正确使用 Babel

1.  利用环境变量合理使用 babel 对不同环境的代码进行处理,例如在开发环境不要引入 polyfill,在开发环境利用 plugin 将动态 import 转为 require 加快打包速度

2.  开启 babel-loader 的缓存加快打包速度

### 总结

最好的学习方式:起一个 webpack 环境,配置不同的.babelrc 观察每次打包后的情况.
这比任何文章都来得实在,多看官方文档,中文总结只是一个参考,具体表现如何还是看实践+文档.

1.  转译方案: @babel/preset-env + @babel/preset-react + 通过命令行升级的 stage-0
2.  polyfill 方案: @babel/polyfill + @babel/preset-env[useBuiltIns:"entry"]
3.  helpers 重复注入解决: @babel/runtime + @babel/plugin-transform-runtime

参考文章及推荐阅读:

- [babel](https://babeljs.io/)
- [大前端的自动化工厂— babel](https://zhuanlan.zhihu.com/p/44174870)
- [babel 7 教程](https://blog.zfanw.com/babel-js/)
- [升级至 babel 7](https://babel.docschina.org/docs/en/v7-migration)
- [再见, babel-preset-2015](https://zhuanlan.zhihu.com/p/29506685)
- [你真的会用 babel 吗？ ](https://github.com/sunyongjian/blog/issues/30)
- [21 分钟精通前端 Polyfill 方案](https://zhuanlan.zhihu.com/p/27777995)
- [babel-polyfill VS babel-runtime VS babel-preset-env](https://juejin.im/post/5aefe0a6f265da0b9e64fa54)

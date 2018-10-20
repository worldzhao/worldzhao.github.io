---
title: 关于几种数据Mock的手段
date: 2018-10-21
tags: [webpack]
categories: 工程化
---

## 数据 Mock

在前后端约定好 api 接口后,同步进行开发,前端没有数据支持,造成了阻塞.

理想情况下定义好 api 路径和数据之间的映射即可,本文重点放在第三种方式.

### [Mock.js](http://mockjs.com/)

Mock.js: 生成随机数据，拦截 Ajax 请求.

使用方法比较简单:

1.  定义好映射关系;
2.  在入口文件引入 Mock.js ;
3.  具体使用建议查看官方文档.

需要注意的是开发环境下才要这样做,可以在起服务的时候设置环境变量从而进行判断.

### [json-server](https://www.npmjs.com/package/json-server)

1.  定义好映射关系;
2.  通过 json-server 起服务,例如:'json-server mock/mock.js --watch --port 8090';
3.  将请求转发到(webpack-dev-server:proxy)json-server 监听端口

当然生成数据的时候还是可以和 Mock.js 一起使用

### [webpack-api-mocker](https://github.com/jaywcjlove/webpack-api-mocker)

webpack-api-mocker 是一个 webpack-dev-server 的中间件,用于为 REST APIs 提供 Mock 数据.

相较于前两种,webpack-api-mocker 基于 webpack-dev-server 提供的 before 钩子进行处理,直接在开发服务器上进行 Mock 操作,我觉得是比较优雅的一种实现方式,当然,还是可以和 Mock.js 结合使用.

- 安装

```
npm install webpack-api-mocker --save-dev
```

- 使用

webpck.dev.config.js

```js
const apiMocker = require('webpack-api-mocker')

module.exports = {
  devServer: {
    // ...
    before(app) {
      apiMocker(app, path.resolve(__dirname, '../mock/index.js'), {})
    }
  }
}
```

- 编写 mock 数据

/mock/index.js

```js
module.exports = {
  'GET /api/user': function(req, res) {
    res.send(Mock.mock({ name: '@cname', intro: '@word(20)' }))
  },
  'POST /api/login/account': (req, res) => {
    const { password, username } = req.body
    if (password === '888888' && username === 'admin') {
      return res.send({
        status: 'ok',
        code: 0,
        token: 'sdfsdfsdfdsf',
        data: { id: 1, username: 'kenny', sex: 6 }
      })
    } else {
      return res.send({ status: 'error', code: 403 })
    }
  }
}
```

webpack-dev-server 本质就是一个 express 服务.既然是 express ,就可以注册路由返回数据,webpack-api-mocker 便是利用 webpack-dev-server 提供的 before 钩子函数对其`app`注册定义的路由并返回 mock 数据.

webpack-dev-server 的文档里在介绍 before 参数时也是用的一个 mock 的例子:),如下:

```js
module.exports = {
  //...
  devServer: {
    before: function(app) {
      app.get('/some/path', function(req, res) {
        res.json({ custom: 'response' })
      })
    }
  }
}
```

现在要做的就是从映射关系表到实现注册路由并绑定相应的回调函数.

好像看[廖雪峰的 js 教程](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001471133885340dad9058705804899b1cc2d0a10e7dc80000)里有类似操作,其实也没怎修改,如下所示:

/config/dev.mocker.js

```js
module.exports = (app, dataPath) => {
  const mapping = require(dataPath)
  for (const url in mapping) {
    if (url.startsWith('GET ')) {
      const path = url.substring(4)
      app.get(path, mapping[url])
      console.log(`register URL mapping: GET ${path}`)
    } else if (url.startsWith('POST ')) {
      const path = url.substring(5)
      app.post(path, mapping[url])
      console.log(`register URL mapping: POST ${path}`)
    } else {
      console.log(`invalid URL: ${url}`)
    }
  }
}
```

这是基于 webpack-dev-server 提供给我们的 before 钩子函数进行的处理,对一般的数据 mock 场景已经足够了.

倘若在转发代理方面需要有大量自定义操作,譬如在请求头设置一些认证信息,转发错误自动采用 mock 数据,有选择性的进行 mock,一个 before 钩子函数可能会显得力不从心.

可以通过编写脚本的方式去打造出个性化的开发服务器环境,主要依赖 webpack/webpack-dev-middleware/webpack-hot-middleware/http-proxy-middleware 等模块以或中间件去丰富请求的处理逻辑,实际上 webpack-dev-server 底层也是这样做的.

有时间丰富一下这部分内容.

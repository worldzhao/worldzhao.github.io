---
title: 关于 git flow 工作流程的一点思考
date: 2018-09-01
tags: [git]
categories: 工作技能
---

# 关于 git flow 工作流程的一点思考

[Git Flow 工作流程](https://www.jianshu.com/p/9a76e9aa9534)

[Git 分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)

具体流程暂不细表，参考文章中已经极为详细。

## Git Flow 工作流程

1.  开发阶段：从 develop 分支拉出 feature 分支进行开发
2.  联调阶段：直接使用 feature 分支进行联调
3.  提测阶段：将 feature 分支合并入 develop 分支，并基于 develop 分支拉出 release 分支进行提测
4.  发布阶段：将 release 分支分别合入 master 分支以及 develop 分支，并打上版本号

### 碰到的问题

倘若存在 2 个 feature 分支[feature1.0/feature.2.0]

feature1.0 已经合并入 develop 分支并拉出 release 分支进行提测 bug 修复

过了不久，feature2.0 开发完毕，合并入 develop 分支并拉出 release 分支进行提测

如果第二个分支先于第一个分支发布，master 就合并进了还没修复完的 feature1.0 的代码

### 解惑

通过第一文章作者留下的联系方式加了他的微信，这是他回复给我的:

```
这种工作方式：一般情况要遵循feature1.0分支先于feature2.0发布。

也就是：只要先合并到develop就说明已经具备先发的条件。
```

同时在第二篇文章阮老师留言下面发现了同样类似的回答：

![git-flow-1.png](https://upload-images.jianshu.io/upload_images/4869616-e96013593a9d91f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![git-flow-1.png](https://upload-images.jianshu.io/upload_images/4869616-e96013593a9d91f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[第二个回答也是我们小团队目前正在使用的方式]

感谢前人。

## 公司目前使用的流程

1.  开发阶段：从 master 分支拉出 feature 分支进行开发
2.  联调阶段：直接使用 feature 分支进行联调
3.  提测阶段：从 master 分支拉出一个 release 分支，使用该 release 合并 feature 分支，然后进行提测
4.  发布阶段：提测结束后，将 release 分支合并入 master 分支，进行发布，也可直接发布 release 分支再合并入 master 分支，并打上版本号

### 好处

需求发布无需严格遵循需求先后发布时间，不存在标准 git flow 工作流程碰到的问题

### 缺陷

在标准工作流中

master 分支用来记录官方发布轨迹；develop 分支是一个集成分支，用来记录开发新功能的轨迹。

而在实际开发过程中由于没有使用 develop 分支，master 分支 commit 较为混乱，使用标准工作流又会碰到发布冲突的问题

- 总结

来自第一篇文章作者(再次表达感谢)：

不管用什么方式，能保证工作顺利进行就行。

每个项目工程的复杂度都不一样，要从实践中找到适合团队协作的方式。

还就是，有些约定，是必须遵循的。

### 后话

最近写博客越发觉得自己只是知识的复述者，复述得还没有别人好，开始怀疑自己花时间投入写博客是否值得。

自己还处于学习阶段，也没有原创输出。

倘若不写，又感觉自己学习的知识印象不深，写还是要写的，权当学习记录了。

当然，倘若能够帮助到一些人，我会更开心。

如果文中有表述不当的地方还望指出。互相学习共同进步。

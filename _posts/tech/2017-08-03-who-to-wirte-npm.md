---
layout: default
title: 如何编写并发布一个npm包
tags: tech
---

## 干啥的

做js开发的 人应该对 npm install xxxx 这个指令并不陌生。当做一个需求的时候，有些功能有人已经写好了，你只需要执行这个命令去安装它的包就可以了。非常方便。那么如何来写一个我们自己的npm包呢？

## 简单说一些

我们想要的一个工具包类似如下的使用过程

1. npm install xxxx --save
2. 代码中引入

```javascript
const xxx = require('xxx');
xxx.someFunc();
```

## 编写

- 模块内部写的东西不细说了。

- 整个模块需要一个入口 比如index.js。

- 入口配置在 package.json 的 main 字段
```json
"main":"index.js"
```

## 发布到github
在 package.json 的 repository 字段配置仓库信息，这里是git作为仓库
```json
"repository": {
    "type": "git",
    "url": "填写你的git仓库地址"
}
```
然后你本地安装的时候只需要 npm install 你的git仓库地址  就可以了

## 发布到npm官方库

> 不详细介绍了，尽量不往官方仓库提交低质量包。自己使用建议还是放在git上

- 注册npm账号
- 本地登录
- npm publish

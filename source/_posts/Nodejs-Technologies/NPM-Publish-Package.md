---
title: 向NPM仓库发布自己的Package
description: 阐述如何使用npm发布自己的Package
...

阐述如何使用npm发布自己的Package

# 创建Nodejs 模块
参考： [https://docs.npmjs.com/getting-started/creating-node-modules](https://docs.npmjs.com/getting-started/creating-node-modules)
# Publish

参考[官方指导](https://docs.npmjs.com/getting-started/publishing-npm-packages)

简洁总结如下步骤：
1. 创建/添加 npm 包仓库的用户到本地，如果已经有用户直接使用命令 `npm login`,  通过`npm whoami` 查看当前用户
2. 检查package.json 内容，编写README.md， 发布 `npm publish`

> 发布前注意npm的镜像，国内很多由于网络问题会设置淘宝的镜像，发布前记得切换到https://registry.npmjs.org， 关于NPM的源可以参考文章：[https://registry.npmjs.org](https://registry.npmjs.org)
# 问题收集

## 问题1 Incorrect username or password
https://github.com/npm/npm/issues/6545
### workaround
删除文件.npmrc 后重试
> 不同系统的文件位置不同， windows系统在当前用户的目录下，比如：`C:\Users\Administrator\`

## 问题2 you must verify your email before publishing a new package
也许当时注册时候没有验证邮箱， 你需要重新验证下，去npm官网发一个验证到邮箱， 然后照着做
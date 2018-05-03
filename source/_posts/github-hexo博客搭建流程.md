---
title: github+hexo博客搭建流程
date: 2018-05-03 11:08:42
categories: 其他
tags:
---

# 利用hexo搭建个人github blog
## 在github上创建repo
- 登录github.com创建repo:<username>.github.io
- 创建hexo和master分支，并将hexo分支设置为默认分支。
- git clone repo到本地


## node.js相关配置
- 下载node.js
- 进入本地repo，依次执行如下命令：
```
$ npm install -g hexo
$ npm install --save hexo-deployer-git
$ hexo init
```
- 这样hexo初始化已经完成。


## hexo配置
- 修改当前repo下的_config.yml文件

  仔细阅读相关说明之后进行更改基础配置
- 修改deploy标签，将hexo与github仓库关联
- 还可以在hexo官网上下载博客主题


## hexo配置置顶博文

```
$ npm uninstall hexo-generator-index --save
$ npm install hexo-generator-index-pin-top --save
```
下载插件后，将在md头中新增一行
```
top: true
```
## github.com和hexo博文发布命令流程
hexo n + "$title"在/source/_posts文件夹中生成md文件，编辑好md文件后，按如下流程执行命令
```
$ git add .
$ git commit -m ""
$ git push origin hexo
$ hexo d -g
```

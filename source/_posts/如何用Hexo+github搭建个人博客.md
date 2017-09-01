---
title: 如何用Hexo+github搭建个人博客
tags: Hexo
categories: 杂项
comments: true
---
## 安装Hexo

### 全局安装hexo-cli

``` bash
$ npm install hexo-cli -g
```

### 创建一个新文件夹，命名为hexo,然后cd到这个目录下

``` bash
$ hexo init
$ npm install hexo-deployer-git --save
$ npm install hexo-server --save
```


### 新建一篇文章

``` bash
$ hexo n "newblog.md"
```


### 生成静态文件

``` bash
$ hexo g
```
## 部署并同步到GitHub
### 设置Git的user name和email
``` bash
git config --global user.name "yourname"
git config --global user.email "youremail"
```

### 生成SSH密钥
``` bash
ssh-keygen -t rsa -C "youremail@gmail.com"
```

在~/.ssh 目录下打开id_rsa.pub 文件，复制里面的内容，
在个人的github设置里面==> SSH and GPG keys ==> New SSH key,随便取一个名字，内容就是刚刚复制的公钥内容，保存
_config.yml最后加
```
deploy:
      type: git
      repo: git@github.com:raymoe/raymoe.github.io.git
      branch: master
```

### 部署

执行下面的命令

``` bash
$ hexo d
```
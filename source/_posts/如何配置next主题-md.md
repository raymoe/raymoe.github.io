---
title: 如何配置next主题
date: 2017-09-01 16:41:34
tags: [Hexo,next主题]
---
## 安装next主题
参见[官方文档](http://theme-next.iissnan.com/getting-started.html)

新版本迁移到 https://github.com/theme-next

git clone https://github.com/theme-next/hexo-theme-next themes/next



### 配置Next

#### 设置语言
改配置文件，注意是改根目录下的_config.yml
```
language: zh-CN
```

#### 设置预览摘要
设置完模式后，读者们会发现，尽管首页显示的是所有文章的列表，但是每一篇文章都显示了所有内容，这样感觉看起来不舒服，这时候可以启用预览摘要模式，在主题配置文件中找到auto_excerpt属性，将enable设置为true ，将length设置为想要预览到的字数，如下所示：
```
auto_excerpt:
  enable: true
  length: 150
```

#### 添加评论功能
NexT主题集成了评论系统，只需要设置相关的属性即可实现功能，其目前支持多说、Disqus、Facebook评论、Hyper评论、网页云跟帖等

使用友言
```
# Support for youyan comments system.
# You can get your uid from http://www.uyan.cc
youyan_uid: uid
```
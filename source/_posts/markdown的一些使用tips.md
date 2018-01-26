---
title: markdown的一些使用tips
date: 2017-10-19 15:57:45
tags: [Hexo,markdown]
mathjax : true
---

### markdown 换行
空格+空格+回车换行

### 输入数学公式
参考了这篇文章[在 Hexo 中完美使用 Mathjax 输出数学公式](http://cn.clanzd.com/mathjax-for-hexo.html)
#### 方法1
在文件头下面插入
```
---
title: markdown的一些使用tips
date: 2017-10-19 15:57:45
tags: [Hexo,markdown]
---
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```
#### 方法2
如果发现渲染还有问题的话，更换 Hexo 的 markdown 渲染引擎，用hexo-renderer-markdown-it-plus 引擎替换默认的渲染引擎 hexo-renderer-marked 即可。

先卸载 hexo-renderer-marked，再安装 hexo-renderer-markdown-it-plus 插件.

``` bash
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-markdown-it-plus --save
```

安装插件后，如果未正常渲染 LaTeX 数学公式，在博客配置文件 _config.yml 中添加  

```
markdown_it_plus:
  highlight: true
  html: true
  xhtmlOut: true
  breaks: true
  langPrefix:
  linkify: true
  typographer:
  quotes: “”‘’
  plugins:
    - plugin:
        name: markdown-it-katex
        enable: true
    - plugin:
        name: markdown-it-mark
        enable: false
```

#### 方法3

其实如果安装了next主题的话，next主题里面已经集成了mathjax 和 katex
<font color=red>可根据需要修改配置文件</font>
``` json
# Math Equations Render Support
math:
  enable: true

  # Default(true) will load mathjax/katex script on demand
  # That is it only render those page who has 'mathjax: true' in Front Matter.
  # If you set it to false, it will load mathjax/katex srcipt EVERY PAGE.
  per_page: true

  engine: mathjax
  #engine: katex

  # hexo-rendering-pandoc (or hexo-renderer-kramed) needed to full MathJax support.
  mathjax:
    # For newMathJax CDN (cdnjs.cloudflare.com) with fallback to oldMathJax (cdn.mathjax.org).
    cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
    # For direct link to MathJax.js with CloudFlare CDN (cdnjs.cloudflare.com).
    #cdn: //cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML
    # For automatic detect latest version link to MathJax.js and get from CloudFlare.
    #cdn: //cdn.bootcss.com/mathjax/2.7.1/latest.js?config=TeX-AMS-MML_HTMLorMML
```

示例：$r=\varphi (N) = \varphi (p)\varphi (q)=(p-1)(q-1)$
---
title: Jekyll笔记
author: huimingz
date: 2021-02-12 16:15:42 +0800
category: [jeklly]
tags: [jeklly]
pin: false
---

自用之前用于搭建Wordpress博客的服务器到期后，就在考虑使用[Hexo](https://hexo.io/)或者[Hugo](https://gohugo.io/)来搭建博客。经过一番考量之后，最终选择和[Jekyll](http://jekyllrb-ja.github.io/)。

使用Jekyll的原因：有很多我喜欢的主题，托管在github，免费。

## 环境配置

先跟着官方文档一套流：[http://jekyllrb-ja.github.io/](http://jekyllrb-ja.github.io/)

使用主题为 [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)，直接跟着文档走就完了（使用的模板项目）。

## 写文档

使用Typora或者Vim，在Typora下图片显示无解，除非使用图床，但免费图床怕哪天跪了，所以图片都放到了资源文件下（压缩后）。

写完文档之后直接push到仓库，自动化构建和发布。

※ ~~有时候会出现缓存导致未显示新文章的问题，强制全局刷新即可。~~ CI&CD结束后稍等一两分钟后便会生效。

## 草稿
在写文章的时候大多数情况下并不会一开始就发布，但又想在本地可以看到，那么这时候就很需要 **草稿** 这个功能。在Jekyll中如何实现草稿功能呢？

### _drafts目录
只需要在项目的根目录下创建一个 ``_drafts`` 目录，将想要作为草稿的文章放到该目录下即可，但这时候执行 ``jekyll s`` 后并不能看到我们的草稿文章，别着急，你需要添加在执行命令的时候添加 ``--drafts`` 参数，示例：

```
$ jekyll s --drafts
```

工作流：
1. 在 ``_drafts`` 目录下新建一个草稿文件，此时的文件名是可以不携带日期信息的，比如： ``post-name.md``（带日期的文件名: ``2021-02-16-post-name.md``）。
2. 如果我想发布这草稿文章，只需要将它迁移至 ``_posts`` 目录下，修改一下发布日期即可。（如果文件名称没有日期，记得添加上）

### published声明
另外一种方法就是在 ``_posts`` 目录下的文件头重声明 ``published: false`` 也可以达到同样的效果，不过这种做法的确定是不能直观的看到哪些文章是待发布的。

当然你可以使用下面的命令来查询哪些文章未发布:

```
$ find ./_posts/ -type f -name "*.md"  | xargs -I {} grep -l  "published: false" {}
```

执行本地预览:

```
$ jekyll s --unpublished
```

当你想要发布这篇文章的时候，只需要修改声明为 ``published: true`` 即可。


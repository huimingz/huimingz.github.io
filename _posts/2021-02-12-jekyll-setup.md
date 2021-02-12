---
title: Jekyll博客的搭建流程记录
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

※ 有时候会出现缓存导致未显示新文章的问题，强制全局刷新即可。

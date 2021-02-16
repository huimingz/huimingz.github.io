---
title: 使用阿里云OSS作为图床
author: huimingz
date: 2021-02-16 12:28:08 +0800
category: [other]
tags: [other]
pin: false
---

这里记录下自己使用阿里云对象存储OSS作为图床的过程。

为什么选择OSS而不是使用一些公共的图床或者直接将图片保存到博客项目中？
1. 公共的图床虽然使用方便，不需要自己维护，但是你不能保证它不会挂掉。
2. 将图片保存到项目中是一种不错的选择，自己维护，不用担心丢失。
3. 使用OSS作为图床只是多一个选择，反正也不贵。

我的方向：今后还是主要将图片直接放到项目中处理，部分图片放到OSS中处理，或者为了方便（文章迁移），所有的图片放到OSS中也是有可能的。

## 准备
1. 阿里云账号
2. [uPic](https://blog.svend.cc/upic/) App

※ 这里的uPic软件主要是为了方便在MacOS下进行图片上传处理，毕竟使用网页版文件管理和oss-browser太麻烦了。如果你是windows用户，可以选择其它合适的app。

## 步骤
1. 购买一个对象存储服务OSS，我这里选择的是标准存储包，40GB存储包。
    ![购买oss](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_09-22-21.png)
2. 购买后需要进行开通，我选择的是流量计费模式，毕竟自己用，估计耗不了多少流量。
    
    关于流量计费价格：
    ![参考价格](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_09-59-56.png)

    详细参考[详细价格总览](https://www.aliyun.com/price/product?spm=a2c4g.11186623.2.7.6cbf2b82dzkmYi#/oss/detail)
3. 新建一个Bucket。
    ![新建Bucket](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_11-52-57.png)
4. 配置权限管理，我这里没有搭配CDN使用，所以读写权限要设置为“公共读写”，配置防盗链和跨域设置（CORS）。
    ![权限配置](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_11-59-51.png)

    ※ 这里面需要注意的是防盗链的提示说明中并没有要求加入scheme，但是这会导致你在本地预览的时候访问图片链接出现403，所以你需要在域名前面添加上scheme（或许）。
5. 添加一个角色账户，后面uPic或者oss-browser需要使用。

    先从上面第4步操作中的“访问控制RAM”下的“前往控制台”进入控制页面，然后新增一个“用户”，我这里只允许了“编程访问”的访问方式，因为我不需要使用这个账户登陆控制台。创建账户期间可能需要你输入短信验证码！
    ![创建账户](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_12-08-08.png)
    
    创建完账户后会弹出一个账户信息页面，务必AccessKey ID和AccessKey Secret，之后需要使用。
    ![账户信息](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_12-11-18.png)

    此时创建的用户并没有OSS的权限，需要给他分配权限。
    ![添加权限1](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_12-16-59.png)
    ![添加权限2](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_12-19-03.png)

5. 最后就是uPic配置以及使用说明，这个直接看[官方文档说明](https://blog.svend.cc/upic/)吧，很简单的。这里需要使用上面提到的AccessKey ID和AccessKey Secret。

## 参考
1. [国内自建图床指南](https://sspai.com/post/61624)


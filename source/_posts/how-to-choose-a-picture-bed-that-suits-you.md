---
title: 如何选择一个适合自己的图床
date: 2020-07-06 23:06:33
tags: tutorial
categories: skill
---

## 前言
因为没有把博客部署在服务器上，而是选择GitHub Pages 的方式，所以如果遇到需要插入图片的时候，只能通过图床来存储图片。

如果不是因为[SM.MS](https://sm.ms/) 图床在今天突然挂掉了，我可能都不会去想是否需要更换图床这个问题。

![](https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/20200706225801.png)

于是我开始寻找一个免费、稳定的图床，最后在众多图床中，最后选择了GitHub 图床。

> 为什么不选择国内的那些图床服务？

我只是想存一些图片，而国内的大部分图床服务，还需要做域名备案以及绑定各种服务，感觉很繁琐，加上我的域名不是在国内的域名服务商那里买的，索性就没有考虑国内的图床服务。

### 图床管理工具
有了图床，就需要顺手配置一个图床管理工具，这里我选择的是 [PicGo](https://github.com/Molunerfinn/PicGo)，仅目前支持的图床就有：SM.MS图床，微博图床，七牛图床，腾讯云COS，阿里云OSS，Imgur，又拍云，GitHub 图床等。

### 创建GitHub 图床
首先，你得有一个[GitHub](https://github.com/) 账号。

#### 1. 新建一个仓库

这个仓库是用于存储图片，最好是public，因为private的仓库，图片链接会带token，而这个token会存在过期的问题。

#### 2. 获取授权token
通过`Settings->Developer settings->Personal access tokens` [创建一个新的token](https://github.com/settings/tokens/new) 用于PicGo操作你的仓库。

把repo的勾打上即可，点击Generate token的绿色按钮生成 token。

创建成功后，会生成一串token，这串token之后不会再显示，所以第一次看到的时候最好保存好。

### 配置PicGo
GitHub 图床的配置还是比较简单的，下面是参数说明。

![](https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/20200706225324.png)

* 仓库名：你的图床仓库的名称，格式为：`username/repository`
* 分支名：一般选择默认分支 `master`
* Token：刚才生成的 Token
* 存储路径：指定存放在仓库的哪个目录下
* 自定义域名：`raw.githubusercontent.com/username/repository/branch`

自定义域名最好按照一定的规则去定义：`raw.githubusercontent.com`+你的github用户名+仓库名称+分支名称

> `raw.githubusercontent.com` 是github用来存储用户上传文件的服务地址，是github 的素材服务器 (assets server)。

通常配置完成之后，就可以直接使用了。

如果你上传失败的情况，可以打开PicGo 的日志看看具体是什么异常

![](https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/20200706220223.png)

如果得到了这样的异常，那么大概率是因为你没有开启全局代理。
```
[PicGo ERROR] RequestError: Error: connect ECONNREFUSED 13.250.168.23:443`
```
因为GitHub 服务器和国内 GFW 的问题会导致有时上传成功，有时上传失败，所以需要自备好科学上网工具。

如果你还有其他问题，可以查阅 [PicGo FAQ](https://github.com/Molunerfinn/PicGo/blob/dev/FAQ.md)。

### 总结
* 如果你和我一样，讨厌域名备案，有希望能有一个免费、稳定的图床，那么一定不要错过GitHub 图床。
* 如果你只是需要存储一些不怎么重要的图片，那么可以使用免费不限大小的SM.MS图床。
* 如果打算长期稳定使用可以优先选择又拍云或者七牛云。
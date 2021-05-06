---
title: 如何利用Jenkins 实现标签或者分支选择性构建
date: 2021-05-06 21:08:51
tags: ["Jenkins"]
categories: ["Jenkins"]
---

如题。

<!-- more -->

需求可以简单描述为：在Jenkins 中通过手动的方式自主选择标签或者分支进行构建。而不是通过 Push 事件进行自动触发。

在正式开始之前，需要先安装 `Git Parameter` 插件。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506205636.png)

在可选插件中搜索`Git Parameter`，进行安装。

正常安装完成，可以看到如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506205903.png)

创建一个自由风格的软件项目：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506205922.png)

选择参数化构建过程，参数类型选择分支或标签：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506210012.png)

源码管理选择Git，填上项目地址，如果是私有项目，需要添加 Credential：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506210049.png)

最后点击保存即可。

点击`Build with Parameters`，可以看到所有标签和分支，手动选择不同的分支和标签即可进行构建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210506210303.png)

可以看到核心的步骤其实只有两步，如果还有其他需求，比如构建完成之后，执行某个脚本，也是可以实现的，

## 参考链接
* [Jenkins教程（三）添加凭据与流水线拉取Git代码](https://www.huaweicloud.com/articles/0591d64c9060a6484280fe1d55d251dc.html)
* [Jenkins参数化构建-插件:Git Parameter](https://www.jianshu.com/p/927a1599f7a0)
* [Jenkins 中使用 Git Parameter 插件动态获取 Git 的分支](http://www.mydlq.club/article/45/)
* [Jenkins：使用Git Parameter插件实现tag或分支的选择性构建](https://www.cnblogs.com/zt007/p/9472524.html)
* [利用 jenkins 达到提 tag 自动打包](https://testerhome.com/articles/17383)
* [Jenkins 实现前端自动打包,自动部署代码及邮件提醒功能](https://www.cnblogs.com/tugenhua0707/p/11949644.html)

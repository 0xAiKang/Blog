---
title: Hexo Volantis 主题优化 | 增加分析与统计
date: 2020-07-09 21:14:37
tags: ["Hexo", "Tutorial"]
categories: Tutorial
---

Volantis 默认支持 [不蒜子](http://busuanzi.ibruce.info/) 的访问统计，可以自行添加[百度统计](https://tongji.baidu.com/)和 [Google Analytics](https://analytics.google.com/)。

<!-- more -->

## 环境要求
* Hexo：4.2
* Node：12
* Volantis：2.6

## 分析与统计

### 字数和阅读时长
1. Volantis 默认没有安装 `wordcount`插件，所以需要手动安装：

```
npm i --save hexo-wordcount
```

2. 修改主题配置文件`themes/volantis/_config.yml`，将 wordcount 插件打开

```

plugins:
  ...
  # 文章字数统计、阅读时长，开启需要安装插件: npm i --save hexo-wordcount
  wordcount: true
```

3. 继续修改主题配置文件`themes/volantis/_config.yml`，将 `wordcount` 放在需要显示的 meta 位置：

```
# 布局
layout:
  on_list:
    meta: [..., wordcount, ...]
  on_page:
    meta:
      header: [..., wordcount, ...]
      footer: [..., wordcount, ...]
```

### 百度统计
百度统计是百度推出的一款免费的专业网站流量分析工具，能够告诉用户访客是如何找到并浏览用户的网站，在网站上做了些什么，非常有趣，接下来我们把百度统计添加到自己博客当中。

1. 访问[百度统计首页](https://tongji.baidu.com/)，注册一个账号后登陆，添加你的博客网站。

![](https://raw.githubusercontent.com/0xAiKang/CDN/master/blog/images/20200709204712.png)

2. 点击获取代码，复制该代码。

3. 在主题配置文件中，增加以下内容：

```
cnzz: true
```
用于设置是否开启百度统计。

4. 在`themes/volantis/layout/_partial`目录下，新建一个`cnzz.ejs`文件，将刚才复制的内容粘贴进去：

```
<% if (theme.cnzz){ %>
<script>
    var _hmt = _hmt || [];
    (function () {
        var hm = document.createElement("script");
        hm.src = "https://hm.baidu.com/hm.js?xxxxxxxxxxxxxxxxxxxxxxx";
        var s = document.getElementsByTagName("script")[0];
        s.parentNode.insertBefore(hm, s);
    })();
</script>
<% } %>
```

5. 最后将以下内容放在网站首页的尾部`themes/volantis/layout/_partial/footer.ejs`中：

```
<%- partial('cnzz') %>
```
完成以上所有操作之后，可以在[百度统计管理页面](https://tongji.baidu.com/sc-web/10000236600/home/site/index)检查代码是否安装正确，如果正确安装，通常二十分钟之后就可以看到网站的分析数据了。
---
title: Hexo Volantis 主题优化 | 添加日历图
date: 2020-07-09 21:07:08
tags: ["Hexo", "Tutorial"]
categories: Tutorial
---

一直觉得GitHub 日历图（代码提交统计样式）很好看，偶然发现是可以通过配置将日历模块引入到Hexo 的主题中的。

默认效果如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200709204832.png)

因为我使用的Hexo 主题是`Volantis`、而该主题目前并没有集成该控件，所以需要手动配置。

<!-- more -->

## 环境要求
* Hexo：4.2
* Node：12
* Volantis：2.6

Volantis 低版本可能会不适用于本文介绍的方法，可以参考 `YINUXY` 的 [Hexo主题美化 | 给你的博客加上GITHUB日历云和分类](https://cloud.tencent.com/developer/article/1597223)

## 配置

1. 在主题配置文件 `themes\volantis\_config.yml` 下添加以下内容：
```
postCalendar: true 
```
用于设置在归档页面中是否显示'文章日历'控件，如果不想显示，设置为 `false` 即可。

2. 在归档页面 `themes/volantis/layout/archive.ejs` 添加以下代码：

```
<div id="calendar">
		<% if (theme.postCalendar) { %>
		<%- partial('_widget/post-calendar') %>
		<% } %>
</div>
```
具体添加位置：

![IMAGE](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200709205348.png)

这里会根据主题配置文件中的`postCalendar`的值，来判断是否需要渲染。

3. 点击下载日历样式文件 [post-calendar.ejs](https://github.com/0xAiKang/CDN/blob/master/blog/js/post-calendar.ejs)，放置于`themes/volantis/layout/_widget`目录下。

将其中的第 16 行，替换成以下内容：

```
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/0xAiKang/CDN@1.0/blog/js/echarts.min.js"></script>
```

至此已经完成了，使用`hexo generate && hexo server`查看是否可以正常加载日历图。

默认的样式是高仿`gittee`，如果觉得不满意，可以参考[官方文档](https://echarts.apache.org/zh/option.html#calendar)自定义。

### 参考链接
* [hexo（sakura）仿gitee添加文章贡献度日历图（echarts）](https://blog.csdn.net/cungudafa/article/details/106420842)


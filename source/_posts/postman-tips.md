---
title: Postman 使用技巧整理
date: 2020-09-19 20:58:16
tags: ["Postman", "JSON", "PHP"]
categories: ["PHP"]
---

[Postman](https://www.postman.com/) 作为http 请求工具，无论是开发还是测试所使用的频率还是挺高的，这篇笔记用来整理一下常用的使用技巧。

<!-- more -->

## 发送表单提交
这里的表单提交就是指传统的表单提交。

核心请求头信息：
```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;
Content-Type: application/x-www-form-urlencoded
```
body 的数据格式选择`form-data`。

## 发送Ajax 请求
核心请求头信息：
```
Accept: application/json, text/javascript, */*;
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
```

body 的数据格式选择 `x-www-form-urlencode`，如果选择`form-data`则接收到的数据格式会是这个样子：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200919202055.png)


如果以`x-www-form-urlencode`格式进行提交，那么接收到的数据是这个样子，可以直接通过魔术变量获取使用。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200919202746.png)

### 如何把请求参数作为json 格式进行提交？

在`Body`中，选择`raw` 然后把请求参数以json 的格式填进去。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200919203537.png)

不过需要注意，以json 格式提交的请求，用常见的魔术变量获取不到，需要使用以下方式：
```
json_decode(file_get_contents('php://input'));
```

## 提交文件
有时候我们希望可以测试文件提交，使用 Postman 当然也可以完成。

请求方式选择POST，Headers 可以不用做选择，Body 选择 `form-data`，类型由默认的text 改成 file，然后选择需要提交的文件即可。

注意：key 最好也填上 file 这个关键字。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201030081600.png)
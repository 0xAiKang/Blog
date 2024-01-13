---
title: PHP 使用 Headless Chrome 生成 PDF
date: 2024-12-25 16:25:03
tags: ["PHP"]
categories: ["PHP"]
---

之前[这篇笔记](https://www.0x2beace.com/php-handles-various-pdf-scenarios/)有记录，如何使用 PHP 操作 PDF，其中生成 PDF 部分，是通过 [tcpdf](https://github.com/tecnickcom/TCPDF) 这个扩展包实现的，虽然可以也能生成 PDF，但是有比较多的限制：
1. CSS 支持级别不一样
2. HTML 支持级别不一样，例如 tcpdf 对于表格的生成，只能使用 table 标签，如果使用 div 标签，样式全部会丢失
3. 中文乱码问题是否有解决方案

在这种情况下，如果需要生成一些比较复杂的 PDF，就会变得十分困难。

今天要介绍的[Chrome PHP](https://github.com/chrome-php/chrome)，则可以完美解决以上问题。

<!-- more -->

## Headless Chrome

Headless Chrome 别名无头浏览器，其实在之前的多篇笔记中，都有提到过。

上一次生成 PDF 时，是打算使用它，但是因为当时时间比较紧，使用过程中遇到比较多的问题，就临时改用了其他解决方案。

这一次因为需要生成的 PDF 比较复杂，使用之前的解决方案都无法解决。

好在这次时间比较充裕，多花了些时间，顺利解决了。

过程中遇到一些问题，在此整理出来。

使用 Chrome PHP 之前，需要安装 Chrome or Chromium，安装和使用就不在此过多介绍了，可以参考这篇笔记。

这篇笔记主要介绍，使用过程中遇到的一些问题。

### 安装 sockets 扩展
最新版本的 Headless，需要安装 sockets 扩展，如果没有该扩展，则不会安装最新版本，默认安装不需要 sockets支持的版本。

而使用较低版本，则会有一些 Bug 存在。

解决方案也很简单，就是安装 sockets 扩展，然后升级最新版本即可。

### 安装语言包
在本地开发环境，因为 Mac 系统默认安装了各种字体文件，对中文的支持友好，生成PDF 时，不会出现乱码的情况。

而在 Linux 环境下，默认是缺少一些中文字体文件的。

这种情况下，直接使用 Chrome PHP 去处理中文字符，则会出现乱码：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240113114530.png)
解决方案：

1. 使用 `fc-list` 命令查看系统的字体文件库，是否有中文字体，不过通过不存在，否则也不会乱码


2. 安装 `ttmkfdir`，如果已经安装了，跳过下一步

3. 如果是 Centos，需要创建字体目录，并给文件夹权限

```bash
$ mkdir /usr/share/fonts/chinese && chmod -R 755 /usr/share/fonts/chinese
```

4. 将所需字体文件，上传至 `/usr/share/fonts/chinese` 目录下

5. 执行以下命令：

```bash
$ ttmkfdir -e /usr/share/X11/fonts/encodings/encodings.dir
```

6. 编辑 `/etc/fonts/fonts.con` 字体配置文件：

```conf
<!-- Font directory list -->

    <dir>/usr/share/fonts</dir>
    <dir>/usr/share/X11/fonts/Type1</dir>
    <dir>/usr/share/X11/fonts/TTF</dir>
    <dir>/usr/local/share/fonts</dir>

    # 增加下面这行配置信息，导入中文字体文件
    <dir>/usr/share/fonts/chinese</dir>
    ...
```

7. 分别执行 `fc-cache`、` fc-cache-64` 命令

8. 再次执行 `fc-list` 命令，查看导入的中文字体包是否存在

如果能看到，字体文件存在，基本上就不会出现乱码了。

测试一下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20240113135428.png)

Headless Chrome 的用法非常多，不仅仅只是生成图片、生成 PDF，这里只是它的冰山一角，还有很多更高级的用法，更多用法查看[Github](https://github.com/chrome-php/chrome?tab=readme-ov-file#usage)。

## 参考链接
* [centos使用chrome-cli、chromium或wkhtmltoimage截图时出现的中文字符乱码的解决方案](https://segmentfault.com/a/1190000018687409)
* [Save image but cannot show the unicode string](https://github.com/chrome-php/chrome/issues/78)
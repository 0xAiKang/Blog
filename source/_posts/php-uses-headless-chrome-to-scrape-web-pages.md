---
title: PHP 使用Headless Chrome 抓取网页
date: 2022-03-10 17:42:57
tags: ["PHP"] 
categories: ["PHP"]
---

# PHP 使用 Headless Chrome 抓取网页
通常使用 PHP 抓取网页数据时，最常见的两种方式就是 `CURL` 和 `file_get_contents`。这样确实可以抓取到数据，但是无法等待等待页面上的 `JS` 加载完成之后才进行数据抓取。

于是便有了使用 [PhantomJS](http://phantomjs.org/)，[SlimerJS](https://github.com/laurentj/slimerjs) 这类解决方案。

今天要介绍的并不是上面两位，而是 Headless Chrome。

<!-- more -->

## 什么是 Headless Chrome
Headless Chrome 是 Chrome 浏览器的无界面形态，可以在不打开浏览器的前提下，使用所有 Chrome 支持的特性运行你的程序。\
相比于现代浏览器，Headless Chrome 更加方便测试 Web 应用，获得网站的截图，做爬虫抓取信息等。

## 如何获取 Headless Chrome
在 Mac 上，Chrome 59 以后的版本均搭载了 Headless Chrome，所以只要保证 Google Chrome 版本不要低于 59 即可。

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version

Google Chrome 99.0.4844.51
```

或者直接在 Chrome 中输入：`chrome://version`，进行查看。

---

Windows 用户找一下 Chrome 的安装路径，通常是：`C:\Program Files (x86)\Google\Chrome\Application\chrome`

Linxu 服务器则使用 `google-chrome`（需要提前安装）

### CentOS 安装 Google Chrome

1. 下载 Chrome 最新的rpm
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
```

2. 安装 Chrome
```
sudo yum install ./google-chrome-stable_current_*.rpm
```

3. 查看当前版本
```
google-chrome -version
```

## 如何在终端中使用
Mac 用户在终端使用之前，建议先绑定别名，使用起来更方便：

```
alias google-chrome="/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome"
```


```
google-chrome --headless --disable-gpu --remote-debugging-port=9222 https://www.github.com 
```

参数说明：
* `--headless`：使用 headless 模式运行 Chrome
* `--disable-gpu`：屏蔽现阶段可能触发的错误
* `--remote-debug-port=9222`：DevTools 远程调试Chrome
* `https://www.github.com`：目标地址

### 保存为截图
`--screenshot` 参数可以将页面内容保存为截图：

```bash
google-chrome --headless --disable-gpu --screenshot --window-size=1280,1696 https://github.com
```

### 保存为PDF
`--print-to-pdf` 参数可以将页面内容保存为PDF：

```bash
google-chrome --headless --disable-gpu --print-to-pdf https://github.com
```

### 打印DOM
`--dump-dom` 参数将 `document.body.innerHTML` 输出：
```bash
google-chrome --headless --disable-gpu --dump-dom https://github.com
```

## 在 PHP 中使用 Headless Chrome

[Chrome PHP](https://github.com/chrome-php/chrome) 是一个可以在 headless 模式下使用 `Chrome/Chromium` 的类库。

安装：
```bash
composer require chrome-php/chrome
```

使用：
```php
use HeadlessChromium\BrowserFactory;

try {
    // chrome/chromium 安装位置
    $browserFactory = new BrowserFactory('/Applications/Google Chrome.app/Contents/MacOS/Google Chrome');

    // starts headless chrome
    $browser = $browserFactory->createBrowser();

    // cr3eates a new page and navigate to an url
    $page = $browser->createPage();
    $page->navigate("https://github.com")->waitForNavigation();

    // 获取浏览器滚动条宽高，用于设置：setViewport
    $evaluation = $page->callFunction(
        'function() {
            var width = document.body.scrollWidth;
            var height = document.body.scrollHeight;
            return [width,height];
         }'
    );

    $value  = $evaluation->getReturnValue();
    $width  = $value[0];
    $height = $value[1];
    $page->setViewport($width, $height)->await(); // wait for operation to complete

    // take the screenshot (in memory binaries)
    $screenshot = $page->screenshot([
        'format'  => 'jpeg', // default to 'png' - possible values: 'png', 'jpeg',
        'quality' => 100, // only if format is 'jpeg' - default 100
    ]);

    // 保存图片
    $screenshot->saveToFile("github.png");
} finally {
    // 关闭浏览器
    $browser->close();
}
```

## 参考链接
* [Chrome-PHP](https://github.com/chrome-php/chrome)
* [Chromium Project](https://www.chromium.org/)
* [初探 Headless Chrome](https://zhuanlan.zhihu.com/p/27100187)
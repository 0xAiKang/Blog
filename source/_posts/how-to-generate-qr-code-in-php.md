---
title: 如何在 PHP 中生成二维码
date: 2022-02-26 16:00:47
tags: ["PHP"]
categories: ["PHP"]
---

# 如何在 PHP 中生成二维码

原生 PHP 无法直接生成二维码，可以借助一些第三方的二维码库。这里使用 [endroid/qr-code](https://github.com/endroid/qr-code)。

<!-- more -->

安装：
```bash
composer require endroid/qr-code
```

首先来了解一下二维码的一些基本特征

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220326100220.png)

这是一张常见的二维码，由以下几部分构成：
* ①：外框
* ②：码眼
* ③：码点
* ④：logo
* ⑤：文字

其中码眼和码点是必要的，其余部分则均是为二维码美化的而添加的，非必要。

## 简单的二维码生成

```php
require "vendor/autoload.php";

use Endroid\QrCode\QrCode;
use Endroid\QrCode\Writer\PngWriter;

// 创建二维码
$qr = QrCode::create("Hello World");
$writer = new PngWriter();
$result = $writer->write($qr);

// 输出二维码（方式一）
$result->saveToFile(__DIR__ . "/qr.png");

// 方式二
// header("Content-Type: " . $result->getMimeType());
// echo $result->getString();

// 方式三
// echo "<img src='{$result->getDataUri()}'/>";
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220326130344.png)

`endroid/qr-code` 提供三种不同的输出模式：
* `saveToFile()`：生成二维码以图片的形式保存在本地
* `getString()`：直接输出在浏览器
* `getDataUri()`：输出二维码图片对应的 Base64

## 二维码美化

```php
<?php
require "vendor/autoload.php";

use Endroid\QrCode\QrCode;
use Endroid\QrCode\Writer\PngWriter;
use Endroid\QrCode\ErrorCorrectionLevel\ErrorCorrectionLevelHigh;
use Endroid\QrCode\Color\Color;
use Endroid\QrCode\Logo\Logo;
use Endroid\QrCode\Label\Label;

    // 创建二维码
$qr = QrCode::create("Hello World")
    // 设置容错等级
    ->setErrorCorrectionLevel(new ErrorCorrectionLevelHigh())
    // 设置内容区域宽高，默认为300
    ->setSize(300)
    // 设置外边距，默认为10
    ->setMargin(10)
    // 设置二维码颜色，默认为黑色
    ->setForegroundColor(new Color(0, 0, 0))
    // 设置二维码背景色，默认为白色
    ->setBackgroundColor(new Color(255, 255, 255));

    // 设置 logo
$logo = Logo::create(__DIR__ . "/logo.png")
    // 设置 logo 大小
    ->setResizeToWidth(70);

    // 设置二维码下方的文字
$label = Label::create("扫一扫")
    // 设置文字颜色
    ->setTextColor(new Color(0, 0, 0));

// 输出二维码
$writer = new PngWriter();
$result = $writer->write($qr, $logo, $label);
header("Content-Type: " . $result->getMimeType());
echo $result->getString();
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220326155401.png)

创建二维码时，除了可以放文本，还可以放入 URL、邮箱、深度链接等等。

## 其他问题

endroid/qr-code 似乎不支持外框、码眼、码点的自定义，具体可以查看这个 [Issue](https://github.com/endroid/qr-code/issues/196)。

## 参考链接
* [generate-qr-code-php](https://code-boxx.com/generate-qr-code-php/)
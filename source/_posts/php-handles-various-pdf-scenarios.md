---
title: PHP 处理各种 PDF 场景
date: 2023-05-02 12:58:54
tags: ["PHP"]
categories: ["PHP"]
---

最近的一个项目，其中有不少和 PDF 有关的需求，涉及到生成 PDF、根据 PDF 生成图片、读取 PDF、拆分 PDF。

<!-- more -->

## PHP 生成 PDF
PHP 想要生成 PDF，已经有了许多比较成熟的扩展包：
* [dompdf](https://github.com/dompdf/dompdf)
* [tcpdf](https://github.com/tecnickcom/TCPDF)
* [phpoffice/phpspreadsheet](https://github.com/PHPOffice/PhpSpreadsheet)
* [laravel-dompdf](https://github.com/barryvdh/laravel-dompdf)
* [knp-snappy](https://github.com/KnpLabs/snappy)
* [laravel-snappy](https://github.com/barryvdh/laravel-snappy)

因为使用框架（ThinkPHP）的限制，和 Laravel 有关的扩展包是用不了了，dompdf、phpspreadsheet、tcpdf 都用过之后，最后选择了 tcpdf。

没有选择 dompdf 的原因是，对中文支持不友好，默认生成包含中文字符的 PDF 会产生乱码，要解决这个问题，需要额外安装字体，整个过程不是很方便，便放弃了使用这个包。

安装：
```bash
$ composer require tecnickcom/tcpdf
```

dompdf 和 tcpdf 本质上，都是通过对 HTML 进行渲染，最终生成 PDF。因为 tcpdf 支持的css太低级了，所有的样式都是以内联的方式去写。

示例如下：
```php
public function generatePDF() {
  $pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);
  $pdf->SetCreator('0xAiKang');
  $pdf->SetAuthor('0xAiKang');
  $pdf->SetHeaderData(PDF_HEADER_LOGO, 40, '', '', array(0,64,255), array(0,64,128));
  $pdf->setFooterData(array(0,64,0), array(0,64,128));

  $pdf->setFooterFont(Array(PDF_FONT_NAME_DATA, '', PDF_FONT_SIZE_DATA));

  $pdf->SetDefaultMonospacedFont(PDF_FONT_MONOSPACED);

  $pdf->SetFooterMargin(PDF_MARGIN_FOOTER);

  $pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM);

  $pdf->setImageScale(PDF_IMAGE_SCALE_RATIO);

  $pdf->SetFont('stsongstdlight', '', 12, '', true);

  // Add a page
  // This method has several options, check the source code documentation for more information.
  $pdf->AddPage();
  // todo 组装订单数据

  $html = <<<EOD
      <div>
    <p
      style="
        font-size: 2rem;
        text-align: center;
        margin-top: 0;
        margin-bottom: 0;
        font-weight: 900;
      "
    >
      {$this->companyCN}
    </p>
    <p
      style="
        font-size: 2rem;
        text-align: center;
        margin-top: 8px;
        margin-bottom: 0;
        font-weight: 900;
      "
    >
      {$this->companyEN}
    </p>
    <p
      style="
        font-size: 1.5rem;
        text-align: center;
        margin-top: 8px;
        margin-bottom: 0;
      "
    >
      {$this->companyAddress}
    </p>
    <p
      style="
        font-size: 1.5rem;
        text-align: center;
        margin-top: 8px;
        margin-bottom: 0;
      "
    >
      {$this->companyAddressEn}
    </p>
    <p
      style="
        font-size: 1.75rem;
        text-align: center;
        margin-top: 8px;
        margin-bottom: 0;
        font-weight: 900;
        border-bottom: 2px solid black;
        padding-bottom: 1rem;
      "
    >
      发票
    </p>
    <div
      style="
        display: flex;
        width: 100%;
        justify-content: space-around;
        align-items: flex-start;
        margin-bottom: 1rem;
      "
    >
      <div style="display: flex">
        <span>致:</span>
        <div style="margin-left: 4px">
          <div style="margin-bottom: 4px">[HK016]何记肉食公司</div>
          <div style="margin-bottom: 4px">香港石塘咀和合街14-20号</div>
          <div style="margin-bottom: 4px">翠景关地下D号铺</div>
          <div style="margin-bottom: 4px">
            <span>电话：2544948</span>
            <span>传真：2817293</span>
          </div>
        </div>
      </div>
      <div>
        <div style="margin-bottom: 4px">列印日期：2023/05/02</div>
        <div style="margin-bottom: 4px">月结日期：2023/01/01至2023/01/31</div>
        <div style="margin-bottom: 4px">付款方式：月结</div>
      </div>
    </div>
    <div style="margin: 0">
      <div style="display: flex; border-bottom: 1px solid black">
        <div style="flex: 1">发票日期</div>
        <div style="flex: 1">发票编号</div>
        <div style="flex: 1">连锁店名称</div>
        <div style="flex: 1"></div>
        <div style="flex: 1; text-align: right">金额</div>
      </div>

      <div style="display: flex">
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">
          2023/1/17
        </div>
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">
          T23011109
        </div>
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">

        </div>
        <div style="flex: 1; text-align: right">HKD</div>
        <div
          style="flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            text-align: right;">
          1,270.00
        </div>

      </div>
      <div style="display: flex">
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">
          2023/1/17
        </div>
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">
          T23011109
        </div>
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all">

        </div>
        <div style="flex: 1; text-align: right">HKD</div>
        <div
          style="flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            text-align: right;">
          1,270.00
        </div>

      </div>
      <div style="display: flex; border-bottom: 2px solid black">
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all"></div>
        <div style="flex: 1; overflow-wrap: anywhere; word-wrap: break-all"></div>
        <div style="flex: 1"></div>
        <div
          style="flex: 1; padding-top: 2; text-align: right; font-weight: bold"
        >
          小计
        </div>
        <div
          style="
            flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            text-align: right;
            border-top: 2px solid black;
            font-weight: bold;
          "
        >
          4,420.00
        </div>
      </div>
      <div style="display: flex">
        <div
          style="
            flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            font-weight: bold;
          "
        >
          本期开票张数：
        </div>
        <div
          style="
            flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            font-weight: bold;
          "
        >
          3
        </div>
        <div style="flex: 1"></div>
        <div
          style="flex: 1; padding-top: 2; text-align: right; font-weight: bold"
        >
          本期应付金额:
        </div>
        <div
          style="
            flex: 1;
            overflow-wrap: anywhere;
            word-wrap: break-all;
            text-align: right;
            border-bottom: 2px solid black;
            font-weight: bold;
          "
        >
          4,420.00
        </div>
      </div>
    </div>
  </div>
  EOD;

  // 这里可以作为印章，但必须是网络路径，不能是相对路径
  $pdf->Image("http://192.168.1.24:6350/logo.png", 30, 180, 75, 75, 'PNG', '', '', true, 150, '', false, false, 1, false, false, false);

  $pdf->writeHTML($html, true, false, true, false, '');

  // todo 如果文件已经存在了，不必重复生成，直接返回图片地址
  $pdfPath = ROOT_PATH . "public/uploads/invoice/{$oceanOrderNo}.pdf";

  // 生成 PDF
  $pdf->Output($pdfPath, 'F');
}
```

最终效果如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230502115658.png)

## PHP PDF 生成图片
因为需求要求下载 PDF 之前，有一个 PDF 的预览图，因此，需要想办法生成一张 PDF 的预览图。

一开始，我想的是使用 Headless Chrome 抓取网页的方式生成图片，但是遇到了一些莫名其妙的问题，加上开发时间比较紧张，就没有耗费太多时间在上面，而是考虑其他解决方案了。

最终确定下来的方案就是，通过 PHP 的 imagick 扩展，来生成图片，核心代码如下：
```php
public function pdf2png($fromPath,$targetPath)
{

    try {
        $img =  new \Imagick();
        $img->setCompressionQuality(100);
        $img->setResolution(120, 120);
        $img->readImage($fromPath);

        $canvas = new \Imagick();
        $imgNum = $img->getNumberImages();
        foreach ($img as $k => $sub) {
            $sub->setImageFormat('png');
            $sub->stripImage();
            $sub->trimImage(0);
            $width = $sub->getImageWidth() + 10;
            $height = $sub->getImageHeight() + 10;
            if ($k + 1 == $imgNum) $height += 10;
            $canvas->newImage($width, $height, new \ImagickPixel('white'));
            $canvas->compositeImage($sub, \Imagick::COMPOSITE_DEFAULT, 5, 5);
        }

        $canvas->resetIterator();
        $canvas->appendImages(true)->writeImage($targetPath);
        return $targetPath;
    } catch (\Exception $e) {
        \think\Log::error($e->getMessage());
        return false;
    }
}
```
使用该方法的提前是安装 imagick 扩展，具体的安装过程就不在这里详细展开了。

另外要正常使用 ImageMagick，还需要安装 Ghostscript 这个命令行工具，否则无法正常使用。

该方法，接收两个参数，分别是目标 PDF 所在路径，以及需要生成的图片的所在路径，如果生成成功，则返回路径，否则失败返回 false。

## 读取 PDF
使用 `smalot/pdfparser` 扩展包，可以读取 PDF 文件的指定页内容。

安装：
```bash
$ composer require smalot/pdfparser
```

这个扩展包使用起来，比较简单，示例如下：
```php
<?php
require_once('vendor/autoload.php');

use Smalot\PdfParser\Parser;

//创建解析器对象
$parser = new Parser();

//解析PDF文件
$pdf = $parser->parseFile('example.pdf');

//获取第二页对象
$page2 = $pdf->getPages()[1];

//获取第二页内容
$content = $page2->getText();

//输出第二页内容
echo $content;
```

## 拆分 PDF
使用 `setasign/fpdi` 扩展包可以用来处理 PDF 文件。可以结合 `setasign/fpdf` 来实现将一个多页的 PDF 文档拆分成多个单页的 PDF 文档。

安装：
```bash
$ composer require setasign/fpdi
```

以下是一个完整示例：
```php
<?php
require_once('vendor/autoload.php');

use setasign\Fpdi\Fpdi;

function splitPdf($inputFile, $outputDir)
{
    // 初始化 FPDI
    $pdf = new Fpdi();

    // 获取 PDF 文件的页数
    $pageCount = $pdf->setSourceFile($inputFile);

    // 循环处理每一页
    for ($pageNo = 1; $pageNo <= $pageCount; $pageNo++) {
        if ($pageNo !== 1) {
            // 读取文件
            $pdf->setSourceFile($filename);
        }
        // 导入当前页
        $templateId = $pdf->importPage($pageNo);

        // 获取当前页的大小
        $size = $pdf->getTemplateSize($templateId);

        // 创建新的 PDF 文档
        $pdf->AddPage($size['width'] > $size['height'] ? 'L' : 'P', array($size['width'], $size['height']));
        $pdf->useTemplate($templateId);

        // 输出单页 PDF 文件
        $outputFile = $outputDir . DIRECTORY_SEPARATOR . 'output_' . $pageNo . '.pdf';
        $pdf->Output('F', $outputFile);
        
        // 重置 PDF 文档，以便处理下一页
        $pdf = new Fpdi();
    }
}

$inputFile = 'path/to/your/multipage.pdf';
$outputDir = 'path/to/output/directory';

// 拆分 PDF 文档
splitPdf($inputFile, $outputDir);

echo 'PDF 文件拆分完成。';
```

## 参考链接
* [TCPDF Examples](https://tcpdf.org/examples/)
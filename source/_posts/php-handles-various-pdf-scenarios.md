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

dompdf 和 tcpdf 本质上，都是通过对 HTML 进行渲染，最终生成 PDF。因为 tcpdf 支持的css太低级了，所有的样式都是以内联的方式去写，并且只能使用 table 标签生成 PDF，如果使用 div 标签，很多样式都会丢失。

示例如下：
```php
public function generatePDF() {
$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false);
  $pdf->SetCreator('Oceanus');
  $pdf->SetAuthor('Oceanus');
  // $pdf->SetHeaderData(PDF_HEADER_LOGO, 40, '', '', array(0,64,255), array(0,64,128));
  $pdf->setFooterData(array(0,64,0), array(0,64,128));

  $pdf->setFooterFont(Array(PDF_FONT_NAME_DATA, '', PDF_FONT_SIZE_DATA));

  $pdf->SetDefaultMonospacedFont(PDF_FONT_MONOSPACED);

  $pdf->SetFooterMargin(PDF_MARGIN_FOOTER);

  $pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM);

  $pdf->setImageScale(PDF_IMAGE_SCALE_RATIO);

  $pdf->SetFont('stsongstdlight', '', 13, '', true);

  // Add a page
  // This method has several options, check the source code documentation for more information.
  $pdf->AddPage();
  //组装订单数据
  $order = [];
  $table = "";
  $cartInfo = [];
  $total = 0;
  foreach ($productInfo as $k => $v){
      $total = bcadd($total, $v['amount'],2);
      $table .= '<tr>
      <td>' . $v['item_number'] . '</td>
      <td colspan="2">' . $v['item_name'] . '</td>
      <td>' . $v['weight'] . '</td>
      <td>' . $v['unit_price'] . '</td>
      <td>' . $v['quantity'] . '</td>
      <td>' . $v['amount'] .'</td>
  </tr>';
  }
  //不足6条数据进行补充防止页面不够长，章盖在空白处
  if(count($cartInfo) < 6){
      for ($i = count($cartInfo); $i <6; $i++){
          $table .= '<tr>
              <td></td>
              <td colspan="2"></td>
              <td></td>
              <td></td>
              <td></td>
              <td></td>
          </tr>';
      }
  }

  $html = <<<EOD
<div style="font-size: 12px;">
<table  cellspacing="3" cellpadding="4">
<tr>
  <th colspan="3"  align="left" style="font-size: x-large;">{$this->companyCN}</th>
  <td style="font-size: small">{$this->companyAddress}</td>
</tr>

<tr>
  <td colspan="3"  align="left" style="font-size: x-large;">{$this->companyEN}</td>
 <td style="font-size: small">{$this->companyAddressEn1}</td>
</tr>

<tr>
  <td colspan="3"></td>
  <td style="font-size: small" align="left">{$this->companyAddressEn2}</td>
</tr>
<tr>
  <td colspan="3"> </td>
  <td style="font-size: small">电话Tel：{$this->companyPhone}</td>
</tr>
<tr>
  <td colspan="3"> </td>
  <td style="font-size: small">传真Fax：{$this->companyFax}</td>    
</tr>
<tr>
  <td colspan="3"> </td>

   <td style="font-size: small">電子郵件 E-mail：{$this->companyEmail}</td>    
</tr>
<tr>
  <td colspan="4" align="center" style="font-size: x-large;">
      發票
      <br>
      INVOICE
  </td>
</tr>
<tr>
  <td colspan="8" >
       <table border="1" cellspacing="0"  cellpadding="6" width="100%" >
           <tr>
           <td>
             <table>
                  <tr>
                      <td>客户名稱：</td>
                      <td>{$to['oceanuid']}</td>
                  </tr>
                  <tr>
                      <td>Cust No</td>
                  </tr>
                  <tr>
                      <td width="80">客户： </td>
                      <td>{$to['nickname']}</td>
                  </tr>
                  <tr>
                      <td>Name,</td>
                  </tr>
                  <tr>
                      <td>地址： </td>
                      <td>{$to['address']}</td>
                  </tr>
                  <tr>
                      <td>Address</td>
                  </tr>
                  <tr><td></td></tr>
                  <tr><td></td></tr>
                  <tr>
                      <td>聯絡人： </td>
                      <td>{$to['username']}</td>
                  </tr>
                  <tr>
                      <td>電話： </td>
                      <td>{$to['mobile']}</td>
                  </tr>
                  <tr>
                      <td>傳真： </td>
                      <td>{$to['fax']}</td>
                  </tr>
             </table>
           </td>
           <td>
              <table>
                  <tr>
                      <td width="80">發票編號： </td>
                      <td>{$oceanOrderNo}</td>
                  </tr>
                  <tr>
                      <td>Invoice No</td>
                  </tr>
                  <tr>
                      <td>發票日期： </td>
                      <td>{$invoiceDate}</td>
                  </tr>
                  <tr>
                      <td>Date</td>
                  </tr>
                  <tr>
                      <td>參考編號： </td>
                      <td></td>
                  </tr>
                  <tr>
                      <td>Ref No.</td>
                  </tr>
                  <tr> 
                      <td>銷售員： </td>
                      <td>{$to['oceanpid']}</td>
                  </tr>
                  <tr>
                      <td>Saleman</td>
                  </tr>
                  <tr>
                      <td>付款方式： </td>
                      <td>{$to['paymenttype']}</td>
                  </tr>
                  <tr>
                      <td>Payment</td>
                  </tr>
                  <tr>
                      <td>頁數：</td>
                      <td>1/1</td>
                  </tr>
                  <tr>
                      <td>Pager</td>
                  </tr>
              </table>
           </td>
          </tr>
       </table>
  </td>
</tr>
<tr>
  <td colspan="8" >
      <table class="pdf-table" cellpadding="5" border="1" align="center" cellspacing="0" width="100%">
          <thead>
          <tr>
              <td>貨品編號</td>
              <td colspan="2">貨品名稱</td>
              <td>淨重</td>
              <td>單價</td>
              <td>數量</td>
              <td>金額</td>
          </tr>
          </thead>
          <tbody>
           {$table}
           <tr>
              <td align="center">合計:</td>
              <td colspan="2" align="left">{$total}</td>
          </tr>
          <tr>
              <td align="center">備註:</td>
              <td colspan="7" align="left">{$otherInfo['remark']}</td>
          </tr>
          </tbody>
      </table>
  </td>
</tr>
</table>
</div>
EOD;

  $logo = cdnurl("/logo.png", true);
  // 在 PDF 中插入图片，图片必须是网络路径
  // 后面还有一系列参数，控制图片的位置，以及透明度、边框等等
  $pdf->Image($logo, 151, 237, 35, 35, 'PNG', '', '', false, 150, '', false, false, 0, false, false, false);

  $pdf->writeHTML($html, true, false, true, false, '');

  // todo 如果文件已经存在了，不必重复生成，直接返回图片地址
  $pdfPath = ROOT_PATH . "public/uploads/invoice/{$oceanOrderNo}.pdf";

  // 生成 PDF
  $pdf->Output($pdfPath, 'F');
}
```

最终效果如下：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230508100935.png)

## PHP PDF 生成图片
因为需求要求下载 PDF 之前，有一个 PDF 的预览图，因此，需要想办法根据已有 PDF 生成一张 对应的预览图。

一开始，我想的是使用 Headless Chrome 抓取网页的方式生成图片，但是遇到了一些莫名其妙的问题，加上开发时间比较紧张，就没有耗费太多时间在上面，于是考虑其他解决方案了。

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
该方法，接收两个参数，分别是目标 PDF 所在路径，以及需要生成的图片的所在路径，如果生成成功，则返回路径，否则失败返回 false。

使用该方法的提前是安装 imagick 扩展，具体的安装过程就不在这里详细展开了。

另外要正常使用 ImageMagick，还需要安装 Ghostscript 这个命令行工具，否则无法正常使用。

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
* [PDF爱好者的在线工具](https://www.ilovepdf.com/zh-cn)
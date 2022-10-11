---
title: 如何避免写出 If-Else
date: 2022-02-27 14:41:32
tags: ["PHP"]
categories: ["PHP"]
---

本文并不肯定或者否定哪一种写法，仅仅为大家提供一些其他的编码思路或者一些值得借鉴的点子。

<!-- more -->

## 为什么要避免使用if-else
`if-else` 通常是一个糟糕的选择，它导致设计复杂，代码可读性差，并且可能导致重构困难。

这里以**表驱动法**为例，来介绍一下为什么要避免使用`if-else`。

**表驱动法**的意义在于逻辑和数据分离，在程序中，添加数据和添加逻辑的方式、成本是不一样的，简单来说：
* 添加数据非常简单，属于**低成本**、**低风险**
* 而添加逻辑是复杂的，属于**高成本**、**高风险**

下面是一段用表驱动法重构之前的`if-else`代码
```php
<?php
function contry_initial($country){
    if ($country==="China" ){
       return "CHN";
    }else if($country==="America"){
       return "USA";
    }else if($country==="Japna"){
      return "JPN";
    }else{
       return "OTHER";
    }
}
```

现在如果需要增加一个国家，继续使用`if-else` 写的话，就等同于增加一个逻辑。
```php
<?php
function contry_initial($country){
    if ($country==="China" ){
       return "CHN";
    } else if ($country==="Korea") {
      return "KR";
    }
    // ...
}
```

如果改成表驱动法就是：
```php
<?php
function contry_initial($country){
  $countryList=[
      "China"=> "CHN",
      "America"=> "USA",
      "Japna"=> "JPN",
    ];

   if(array_key_exists($country, $countryList)) {
       return $countryList[$country];
   }
   return "Other";

}
```

改写成表驱动法之后，如果需要增加一个国家，就只需要在数组里面加个数据，此时就分离了数据与逻辑的关系了。

重构到此结束，这样做的好处有哪些呢？

### 代码本身的优势

* 逻辑和数据分离，两者一目了然
* 关系表可以更换，比如国家表格可以是多语言的，中文版表格，英文版表格等，日语版表格，以及单元测试中，可以注入测试表格。
* 在单元测试中，逻辑必须测试，而数据无需测试。可以想象如果没有表格法，弄个多语言，要写多少语句。

### 数据来源的灵活性
国家表格的数据可以来源以下渠道：
* 来自代码
* 来自配置
* 来自数据库
* 来自第三方 API
* ...

### 数据输入修改的成本与风险
我们想想，聘用一个不懂编程，但培训一下就会用后台的客服便宜，还是会一个懂系统开发人员便宜？

如果这个是数据，是来自于数据库的，那么基本上公司的任何有权限的人在后台把这个映射表填一下，就能正常工作了。这个耗费与风险几乎可以忽略不计。 如果数据来自第三方 API，如果第三方添加修改了数据，你也是基本放心的。

但是如果这个是逻辑本身，那么只能是这个系统开发人员进行修改，构建，然后经过一系列的测试，进行专业部署流程，使得这个功能在产品上运行，是个耗费与风险是不言而喻的。另外考虑到多人开发，开发风格不统一的话，那么代码审查就不可避免了。

### 数据格式的强制性和代码风格的随意性
在现实中，多人开发同一个功能是很常见的，这里就会有一个多人代码风格统一的问题。

而对于数据来说，一旦数据的格式被确认之后，数据格式就是强制性的了。

比如在上面的例子中，无论是谁，加几个美国的数据也只能这样加：
```php
 $countryList=[
      "China"=> "CHN",
      "America"=> "USA",
      "Japan"=> "JPN",
      "US"=> "USA",
      "United States of America"=> "USA",
      "美国"=> "USA",
    ];
```
就算原始数据花样丰富，最终数据必须格式化成如此。

然而如果是逻辑的话，人一多，逻辑方法就可能发生变化。你可能指望对方这样写代码：

```php
    if ($country==="China" ){
       return "CHN";
    }else if($country==="America"){
       return "USA";
    }else if($country==="Japan"){
      return "JPN";
    }else if($country==="US"){
       return "USA";
    }else if($country==="United States of America"){
      return "USA";
    }else if($country==="美国"){
       return "USA";
    }else{
       return "OTHER";
    }
```

然而，对方可能会这样写：
```php
 if ($country === "China") {
     return "CHN";
 } else if (in_array($country, ["America", "US", "United States of America", "美国"])) {
     return "USA";
 } else if ($country === "Japan") {
     return "JPN";
 } else {
     return ""
}
```

后来多了一个日本的需求，又交给不同的人去完成，说不定最后会写成这样：
```php
 if ($country === "China") {
     return "CHN";
 } else if (in_array($country, ["America", "US", "United States of America", "美国"])) {
     return "USA";
 } else if ($country === "Japan"||$country === "日本") {
     return "JPN";
 } else {
     return ""
}
```

怎样写，都没有错，然而风格却大相径庭。在多人合作编程过程中，无法控制所有人的风格，如果需要统一风格，必须依靠代码审查，这需要大量资源和成本。

另外，就是因为如此，所有和 if else 相关的逻辑在单元测试中必须进行一次测试，才能有代码覆盖率；而如果是数据，由于数据格式的可控性，无需对数据进行测试。所以说：

* 在多人开发的项目中，逻辑无序而不可控；数据格式有序而极易控制
* 在单元测试中，逻辑区块必须进行测试；而数据本身无需测试

## 提前返回
这也许是初级开发人员很容易犯的错误之一。

```php
public function sayHi($name){
  if (!empty($name)){
    return "Hi" . $name;
  } else {
    // do something else
  }
}
```

只需要删除完全不必要的 Else 块，即可简化此过程：
```php
public function say($name){
  if (!empty($name)){
    return "Hi" . $name;
  } 
  
  // do something else
}
```
像这种条件单一的场景，满足某个条件的情况下执行某些操作并立即返回（提前`return`）是很好地解决方案。

## 表驱动法

下面的案例是另外一个比较常见的：
```php
public function doSomething($input){
  if ($input == 0){
    $gender = "woman";
  } else if ($input == 1){
    $gender = "man";
  } else {
    $gender = "unknown";
  }
  
  return $gender;
}
```

像这样的`If-Else` 语句，可以看到`key` 和`value` 是一一对应的，所以可以这样写：

```php
public function doSomething($input){
  $map = [
      0 => "woman",
      1 => "man",
      // ...
  ];
 
  return !empty($map[$input]) ? $map[$input] : "unknown";
}
```


## 参考链接
* [caoglish—把 if-else 的代码风格改成表格驱动法的意义](https://learnku.com/laravel/t/2712/the-meaning-of-the-if-else-code-style-into-the-table-driven-method)
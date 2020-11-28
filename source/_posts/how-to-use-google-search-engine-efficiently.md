---
title: 如何高效的利用谷歌搜索引擎
date: 2020-11-28 17:03:14
tags: ["Skill","Google Search"]
categories: ["Skill"]
---

整理这篇笔记的目的是整理那些不太常用但又十分有用的Google 搜索引擎搜索技巧。

<!-- more -->

### 搜索完全匹配的搜索结果
有时候我们会有这样一种需求：我需要查找某个关键字同时出现的内容，该怎么做呢？
这个时候就需要用到完全匹配这招了。

在关键字的左右两边分别加上`"`英文状态的双引号，如：
```
"HHKB 是什么"
```

### 从搜索结果中排除特定词
为了进一步筛选搜索结果，还需要学会另一招，利用`-`减号排除特定关键字：

```
"the most important benefit of education"-"unitedstates"
```
上面这段表示的意思是：要求Google 返回含有"the most important benefit of education" 但不存在"unitedstates"的内容。

```
daddy -film
```
daddy 的意思是父亲，同时也是一部电影，当你搜索"daddy" 时，谷歌只返回有关电影的内容。如果你只想搜索时关于父亲，要排除电影，在需要排除的前面加上`-`，例如上面所示。你会发现结果中没有与电影有关的内容。

### 搜索通配符或未知字词
怎样用？

即搜索字符串中可以包含星号`*`，用星号来替代任意字符串。
```
powerful*life
```

### 搜索社交媒体
当你只想在某个社交媒体里找到相关字词时，在用于搜索社交媒体的字词前加上`@`，例如：
```
@twice
```

### 组合搜索
在各个搜索查询字间加上“OR”关键字，例如：
```
race OR marathon
```
搜索到的结果会返回关于 race 或者 marathon，或两者均有的相关内容。

### 搜索特定价格
用这个方法来搜索特定价格的商品，例如想要搜索价格为`$200`的书包，可以这样搜索：
```
$200 bag
```

### 在某个数字范围内执行搜索
比如想要搜索介于 $100 - $200 之间的商品，或者是 10kg - 20kg 的某种东西，亦或者是 1900 - 1945 年发生的事情，等等。

在两个数字之间加上`..`符号，例如搜索价格 $50 - $100 的桌子：
```
amazon table $50..$100
```

### 搜索特定网站
只在特定的网站里搜索相关资料，在相应的域名前面加上`"site:"`，例如要在 youtube 里找关于猫的电影，可以这样搜索：

```
site:youtube.com cat
```

### 搜索相关网站
想找和某个网站有关系或者相似特质的网站，在已知网址前面加上`related:`，例如：
```
related:google.com
```
google.com 是一个搜索网站，加上`related:`关键字之后，搜索的结果是其他搜索引擎，如 Yahoo、Bing 等

### 寻找主题标记
在关键字前面加上`#`符号，

### 获取网站的相关资料
如果你想知道某个网站是关于什么的，可以这样子搜索：
```
info:baidu.com
```

### 多组合运用
1. 在 channelnewsasia.com 网站里搜索关于天灾的意外，除了地震，发生在2012年到2016年之间。

```
site:channelnewsasia.com ~accident "natural disaster" -earthquake 2012..2016
```
其中波浪符号`~`表示也搜索和这个字有关联的内容，如 failure，crash、mishap 等

2. 从两个购物网站搜索手表，价格在 $100 到 $200 之间
```
site:shopee.com.my OR site:amazon.com watch $100..$200
```

3. 从ebay 与 amazon网站搜索苹果与微软的产品，排除平板电脑
```
site:ebay.com OR site:amazon.com apple OR microsoft -tablet
```

4. 在吉隆坡一带搜索低收费住宿，价格在$100 到 $200 之间，排除 airbnb，靠近轻快地铁
```
KL ~budget~accommodation $100..$200 -airbnb "nearby LRT station"
```
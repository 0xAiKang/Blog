---
title: 记录一次网站无法访问事件
date: 2022-05-14 16:53:29
tags: ["一些经验"]
categories: ["一些经验"]
---

最近经历了一次大面积网站无法访问事件，过程比较魔幻，记录一下。

<!-- more -->

五月十二日晚上六点半，刚刚上线了一个版本，今天是自己生日，提前预定了一个蛋糕，准备下班了。

收拾好桌面，左脚都已经离开工位了，突然被同事叫住，被告知网站怎么打不开了，我想都没想就说，你确定吗？我刚刚都还打开过。

他表情凝重的告诉我，是真的。

于是我熟练地打开相关客户端，只见满屏的“网络异常”，此时我才意识到可能是真的出大问题了。

因为两分钟前，我刚上线了一个版本，所以第一时间我以为是是不是我误操作了什么造成的，一下子就慌了。

WebService、站点、DB依次过了一遍相关的日志，没有发现任何异常，此刻我更不安了，因为找不到问题的问题，往往是最难解决的。

同一个服务器下面的其他站点都是正常的，慌乱之中，有想过是否是域名过期了，排查之后发现域名并没有过期。

在经过长达半个小时的排查之后，发现竟然是网站没有备案，导致整个站点被停了...

有些事情往往就是这么巧。

为了避免以后再次遇到类似的问题，整理一下网站无法访问的常见排查思路。

## 问题描述
网站的访问与域名的状态、域名实名认证状态、网站备案状态、解析是否生效、网站网络环境等多个环节有关系。在这些环节中，任意一个环节出现问题，都会导致网站无法访问。

## 排查思路
* 查询域名注册信息：检查域名是否过期、状态是否正常
* 查看域名解析是否生效：检查域名解析是否生效
* 查看域名解析配置：检查域名解析配置是否正确
* 查看域名备案状态：对于部署在中国大陆区域的网站，检查是否通过备案审核
* 查看网站配置：检查网站本地网络环境、网站服务器配置是否正常
* 提交工单

## 查询域名注册信息
通常通过 whois 平台，可以查询域名的注册商、注册周期、状态、DNS服务器等注册信息。

通过这些信息可以快速判断，网站无法访问是否与域名有关。

常用 whois 查询平台：
* [whois查询——站长之家](https://whois.chinaz.com)
* [whois查询——中国万网](https://whois.aliyun.com)
* [whois查询——华为云](https://www.huaweicloud.com/whois/index.html)
* [whois查询——阿里云](https://www.alibabacloud.com/zh/whois)

## 查看域名解析是否生效
通过域名成功访问网站的其中一个条件是，域名到IP地址的解析生效。因此，检查域名解析是否成功，是必不可少的一步。

使用查询命令检测是否生效
* ping
* nslookup

## 查看域名解析配置
解析记录配置错误会导致无法将域名解析到正确的IP地址，从而导致网站无法访问。

这一步往往需要登录到对应的服务器的控制台进行查看，此处就跳过了。

## 查看域名备案状态
对于服务器部署在中国大陆区域的网站，如果未进行备案，或者备案审核未通过，则会导致网站访问网站被阻断。

常用备案查询平台：
* [ICP/IP地址/域名信息备案](https://beian.miit.gov.cn/#/Integrated/index)
* [网站备案查询——站长工具](https://icp.chinaz.com)
* [ICP备案查询网](https://www.beianx.cn)

## 查看网站配置
若域名状态正常、解析生效、网站备案审核通过，网站仍然无法访问，需要进一步查看网站的本地网络以及网站的服务器配置。

这一步则是检查是否是因为服务器自身原因而导致不可用，原因较多，这里只列举几个常见的方向：
1. 被防火墙拦截
2. Web Service异常
3. 本地网络故障
4. 等

## 提交工单
如果上述检查全部没有问题，网站仍然无法访问，可以尝试联系对应的服务商，通过提交工单寻求帮助。

## 参考链接
* [网站无法访问排查思路](https://support.huaweicloud.com/dns_faq/dns_faq_140401.html)
* [怎样测试域名解析是否生效？](https://support.huaweicloud.com/dns_faq/dns_faq_015.html)

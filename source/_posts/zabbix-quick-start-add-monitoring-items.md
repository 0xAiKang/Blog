---
title: Zabbix 快速上手——添加监控项
date: 2020-09-09 21:50:15
tags: ["Zabbix", "Linux"]
categories: ["Linux", "Zabbix"]
---

在Zabbix 默认的监控项中，唯独没有网络状态的监控，而网络状况的监控又是我最关心的，所以需要自己手动添加。

下面介绍的方式仅适合主机数量不多的情况手动添加，如果主机数量很多，使用这种方式会很繁琐低效。

<!-- more -->

至于更好的方式是怎样的，暂时还没有发现。

## 添加监控项
打开`Configuration->Hosts` 主机页面，点击需要监控项的主机的 `Application`。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908202935.png)

在`Application`列表中，如果没有看到 `Network interfaces`这一项，那么可以点击右上角的`Create Appliction`自己创建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908203206.png)

创建完成之后，`items` 默认是没有的，需要我们自己添加，继续点击`items->create items`。

接下来是最重要的一步，添加监控项的具体信息。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908203720.png)

需要注意的地方有下面几个：
* Name：自定义该项监控的名称
* Key：`net.if.in[eth0,bytes]`，其中`eth0`并不是固定的，这个具体的值是被监控得主机得实际网卡。
* Units：`bps`
* Update interval：自动更新时间，这个可以自定义。
* Applications：选择 `Network interfaces`

> 如何确定网卡地址？

进入服务器，输入`ifconfig`命令查看，通常排在最前面得就是实际网卡。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908205948.png)

完成之后，点击`Add`添加监控项。

如果一切顺利的话，可以在刚才添加的监控项列表中看到监控项状态是启用的。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908204614.png)

这个时候已经可以看到该监控项相关的数据了，如果希望在Grafana 中展示，那么只需要在选择Application时，选择`Network interfaces`就好了。

结合Grafana，最后的效果大概是这样：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200908205451.png)

这里只是举了一个典型的例子来了解Zabbix 如何手动添加监控项，其他类型的数据也是通过类似的方式进行添加。

### 参考链接
* [zabbix监控网络的出入口流量](https://www.cnblogs.com/smail-bao/p/6109882.html)
* [Cannot find information for this network interface in /proc/net/dev](https://pdf-lib.org/Home/Details/3901)
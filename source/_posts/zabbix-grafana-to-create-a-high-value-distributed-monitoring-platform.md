---
title: Zabbix + Grafana 打造高颜值的分布式监控平台
date: 2020-09-08 21:08:25
tags: ["Zabbix", "Linux"]
categories: ["Linux", "Zabbix"]
---

在前面了解了如何部署 Zabbix，众所周知Zabbix 的部署并不是难的部分，配置才是最难的那部分。

所以如何获取到想要的那部分数据，将那部分数据以更直观的方式展现出来，这才是我们更关心的。

Zabbix 默认有自己的 Graphs，但是并不好用，所以使用Zabbix + Grafana 打造高颜值的分布式监控平台才是最好的选择。

<!-- more -->

## Grafana 是什么？

> Grafana是一个跨平台的开源度量分析和可是化的工具，可以通过该将采集的数据查询然后可视化的展示，并及时通知。

Grafana 有以下特点：
1. 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式.
2. 数据源：Graphite、InfluxDB、OpenTSDB、Prometheus、Elasticsearch、CloudWatch和KairosDb、Zabbix等。
3. 通知提醒：以可视方式定义最重要指标的报警规则，Grafana将不断计算并发送通知，在数据达到预设阈值时通过slack，PagerDuty等处理通知。
4. 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源。
5. 注释：使用来自不同数据源的丰富事件来展示图表，将鼠标悬停在事件上会显示完整的事件元数据和标记。
6. 过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。

### 安装
Grafana 的安装还是建议根据自己实际的系统环境去[官网](https://grafana.com/grafana/download/7.0.0)选择适合自己的下载链接。

比如我的环境是 Ubuntu 18.04，我想安装 Grafana 7.0，所以我的安装方式应该是：
```
$ sudo apt-get install -y adduser libfontconfig1
$ wget https://dl.grafana.com/oss/release/grafana_7.0.0_amd64.deb
$ sudo dpkg -i grafana_7.0.0_amd64.deb
```

### 启动服务

以守护进程的方式启动 `grafana-server`：
```
$ sudo systemctl daemon-reload
$ sudo systemctl start grafana-server
```

设置开机启动：
```
$ sudo systemctl enable grafana-server.service
```

查看 `grafana-server`所监听的端口：
```
$ sudo netstat -lntp
tcp6       0      0 :::3000                 :::*                    LISTEN      17194/grafana-serve
```
3000 是Grafana 默认监听端口，然后通过浏览器访问 `http://your_ip_address:3000` 即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907174006.png)

正常应该可以看到该页面，如果你能看到3000 端口被监听，但是页面一直打不开，那可能是因为防火墙没有允许3000 端口。

默认的用户名和密码都是：admin，登录之后记得第一时间修改默认密码。

### 安装Zabbix 插件
打开Grafana 的插件列表，找到[Zabbix](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app)。

这里根据实实际情况，选择对应的版本。

通过`grafana-cli` 安装zabbix 插件，将下面这行代码放在安装了 Grafana 的服务器上执行：
```
$ grafana-cli plugins install alexanderzobnin-zabbix-app
✔ Installed alexanderzobnin-zabbix-app successfully
```

安装完成之后，重启Grafana：
```
$ sudo systemctl restart grafana-server
```

然后打开Grafana 的Web 界面，在插件列表中找到 Zabbix。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907175149.png)

点击启用。

### add data source
自从 Grafana 7.0 以后，没有签名的插件默认在 datasource 中是不可见的...

坑啊，最初我安装的是 Zabbix5.0，然后看见Grafana 7.0 好像只适配4.0，心想完了，该不会出现什么版本不兼容的问题吧？

结果在`add data source`这一步，一直找不到 zabbix...

然后今天把5.0 完全卸载了，重新装回了4.0，结果到了`add data source`这一步才发现，还是找不到zabbix，当时心态就崩了...

直到我看见[这篇文章](https://sbcode.net/zabbix/grafana-zabbix-plugin/)，这么重要的信息，官方文档中居然没记录。

如果你无法访问，也可以直接进行修改：
```
# vim /etc/grafana/grafana.ini

# 添加一行
allow_loading_unsigned_plugins = alexanderzobnin-zabbix-datasource
```

然后重启Grafana：
```
$ sudo systemctl restart grafana-server
```

再次打开Web 页面，现在就能找到 Zabbix 了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907181658.png)

### 配置 data source
只用修改以下四个地方就好了，然后点击保存。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907203549.png)

### add dashboard

依次点击`add dashboard-> add new panel`，然后按照以下方式配置，就可以选择展示自己想要的数据了。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907204758.png)

最后的效果：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200907204326.png)

这里只是介绍了 Zabbix + Grafana 最基础的用法，能看到的数据也是最简单的一些，如果想看到更多的数据，那就得更加了解 Zabbix 了。

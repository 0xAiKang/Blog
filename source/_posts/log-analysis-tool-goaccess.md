---
title: 日志分析工具 - GoAccess
date: 2020-08-27 22:59:41
tags: ["GoAccess", "Nginx", "Logs"]
categories: ["Nginx"]
---

日志的重要性不言而喻，可我似乎完全忽略了它，导致往往出现什么问题，第一时间并不是去看日志。

<!-- more -->

很显然我完全忽视了它的强大性，就拿 nginx 的访问日志来说，可以从中分析出如下信息：
1. 请求的响应时间
2. 请求达到的后端服务器的地址和端口
3. 请求是否存在缓存配置
4. 请求体、请求头、响应体和响应头的大小等
5. 客户端的IP 地址、UserAgent 等信息
6. 自定义变量的内容

通过这些信息，可以得到响应耗时的请求以及请求量和并发量，从而分析并发原因，这对于应用级别的服务来说是非常重要的。

## GoAccess 是什么
GoAccess 是一个开源的**实时网络日志分析器**和**交互式查看器**，可以在类 Unix 系统中的终端或通过浏览器运行。 —— GoAccess 官方

> 为什么选择 GoAccess？

1. 因为GoAccess 被设计成一个基于终端的快速日志分析器。它的核心思想是实时快速分析和查看Web服务器统计信息，而无需使用浏览器。同时也可以将输入到HTML 或者 CSV、JSON。

2. GoAccess几乎可以解析任何Web日志格式（Apache，Nginx，Amazon S3，Elastic Load Balancing，CloudFront等）。只需要设置日志格式并根据您的日志运行它。

## GoAccess 入门
昨天在使用 GoAccess 时，踩到了一些坑，导致我一度认为这个工具是不是存在什么Bug。因为在看别人的教程中都是开箱即用。

下面从安装到使用会一一详细说明。

### 安装 GoAccess
因为服务器的操作系统是 `Ubuntu`，所以这里以 `Ubuntu`为例：

 因为并非所有发行版都提供最新版本的 GoAccess，所以这里使用官方提供的最新稳定版的安装方式
```
$ echo "deb http://deb.goaccess.io/ $（lsb_release -cs）main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
$ wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add - 
$ sudo apt-get update
$ sudo apt-get install goaccess
```
### 确定日志格式
在计算机安装了GoAccess 之后，要做的第一件事情就是确定访问日志的日志格式，可以在永久设置它们，也可以通过命令行传递他们。

这里用Nginx 的 access.log 为例
```
36.113.128.155 - - [28/Apr/2019:02:20:01 +0000] "GET /Manage/Dingdan/fail_index/startTime/2019-04-28+00%3A00%3A00/endTime/2019-04-28+23%3A59%3A59.html HTTP/1.1" 200 7798 "http://www.692213.com/Manage/Dingdan/fail_index/startTime/2019-04-28+00%3A00%3A00/endTime/2019-04-28+23%3A59%3A59.html" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36"
```

方式一，配置`.goaccessrc`文件：
```
vim ~/.goaccessrc

time-format %T
date-format %d/%b/%Y
log_format %h %^[%d:%t %^] "%r" %s %b "%R" "%u" %^
```

方式二，在命令行传递参数：
```
$ goaccess nginx/access.log --log-format='%h %^[%d:%t %^] "%r" %s %b "%R" "%u" %^' --date-format=%d/%b/%Y --time-format=%T
```

> 注意：无论是配置文件还是命令行参数 都不是永远不变的，只是相对于你要监控的日志格式。

### 运行GoAccess
方式一，通过`-p`参数，指定配置文件。
```
$ goaccess nginx/access.log  -p ~/.goaccessrc
```

方式二，直接在命令行参数中指定日志格式，详情见上面的例子。

#### 终端输出
以下提示使用预定义日志格式的日志配置对话框供您选择，然后实时显示统计信息。
```
$ goaccess nginx/access.log -c
```

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827144401.png)

通常选择第三个，通用日志格式（CLF），成功之后就是这样个样子：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827151540.png)

控制台下的操作方法：
```
* F1或h主要帮助。
* F5重绘主窗口。
* q退出程序，当前窗口或折叠活动模块
* o或ENTER展开所选模块或打开窗口
* 0-9并将Shift + 0所选模块设置为活动状态
* j在展开的模块中向下滚动
* k在扩展模块中向上滚动
* c设置或更改方案颜色
* ^ f在活动模块中向前滚动一个屏幕
* ^ b在活动模块中向后滚动一个屏幕
* TAB迭代模块（转发）
* SHIFT + TAB迭代模块（向后）
* s对活动模块的排序选项
* /搜索所有模块（允许正则表达式）
* n找到下一个出现的位置
* g移至屏幕的第一个项目或顶部
* G移动到屏幕的最后一项或底部
```

#### 静态HTML 输出
以下内容分析访问日志并在静态HTML报告中显示统计信息。
```
$ goaccess -a -d -f nginx/access.log.1 -p ~/.goaccessrc -o /var/www/report.html
```

#### 实时HTML 输出
```
$ goaccess -a -d -f nginx/access.log.1 -p ~/.goaccessrc -o /var/www/report.html --real-time-html
```
然后用浏览器访问，大概就是这个样子：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20200827151617.png)

#### 配置文件及日志格式说明
GoAccess 的配置文件位于`%sysconfdir%/goaccess.conf`或`~/.goaccessrc`

> 其中，%sysconfdir%是 /etc/，/usr/etc/ 或 /usr/local/etc/

`time-format`和`date-format`的格式通常都是固定的，只有`log-format`的格式视具体日志格式而定。
```
time-format %T

date-format %d/%b/%Y

```

`log-format `常用格式说明：
```
* %x与时间格式和日期格式变量匹配的日期和时间字段。当给出时间戳而不是日期和时间在两个单独的变量中时使用。
* %t时间字段匹配时间格式变量。
* %d与日期格式变量匹配的日期字段。
* %v服务器名称根据规范名称设置（服务器块或虚拟主机）。
* %e这是HTTP身份验证确定的请求文档的人的用户标识。
* %hhost（客户端IP地址，IPv4或IPv6）
* %r来自客户端的请求行。这需要围绕请求的特定分隔符（单引号，双引号等）可解析。否则，使用特殊的格式说明符，如组合%m，%U，%q和%H解析各个字段。
注意：使用或者%r获得完整的请求OR %m，%U，%q并%H形成你的要求，不要同时使用。
* %m请求方法。
* %U请求的URL路径。
注意：如果查询字符串在%U，则无需使用%q。但是，如果URL路径不包含任何查询字符串，则可以使用%q并将查询字符串附加到请求中。
* %q查询字符串。
* %H请求协议。
* %s服务器发送回客户端的状态代码。
* %b返回给客户端的对象大小。
* %R“Referer”HTTP请求标头。
* %u用户代理HTTP请求标头。
* %D服务请求所需的时间，以微秒为单位。
* %T服务请求所需的时间，以毫秒为单位，分辨率为毫秒。
* %L 服务请求所用的时间，以毫秒为单位的十进制数。
* %^忽略此字段。
* %~向前移动日志字符串，直到找到非空格（！isspace）char。
* ~h X-Forwarded-For（XFF）字段中的主机（客户端IP地址，IPv4或IPv6）。
```

#### 常用参数
* `-f`：指定需要分析的日志文件路径
* `-c`：程序启动时提示日志/日期配置窗口
* `-p`：指定要使用的自定义配置文件
* `-d`：在HTML或JSON输出上启用IP解析器
* `-o`：输出到指定扩展名文件中（Html、Json、CSV）
* `-a`：按主机启用用户代理列表。为了更快地解析，请不要启用此标志
* `-d`：在HTML或JSON输出上启用IP解析器。

总结：GoAccess 从安装到使用还是非常方便的，不仅可以对历史的日志进行分析，也能实时对日志进行分析，所支持的日志格式基本能满足大多数应用场景。

### 参考链接
* [GoAccess 官网](https://goaccess.io/)
* [GoAccess 入门](https://goaccess.io/get-started)
* [使用GoAccess 分析Nginx 日志](https://www.jianshu.com/p/c6310332f411)
* [将Nginx log_format转换为goaccess配置文件](https://github.com/stockrt/nginx2goaccess)
* [GoAccess 日志格式转换案例一](https://serverfault.com/questions/779405/goaccess-date-time-log-format-error)
* [GoAccess 日志格式转换案例二](https://github.com/allinurl/goaccess/issues/1244)
* [GoAccess 日志格式转换案例三](https://github.com/allinurl/goaccess/issues/668)
* [GoAccess 日志格式转换案例四](https://github.com/allinurl/goaccess/issues/1338)

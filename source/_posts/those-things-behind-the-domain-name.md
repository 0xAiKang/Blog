---
title: 『转载』域名背后那些事
date: 2021-04-11 15:14:30
tags: ["域名"]
---

互联网中的地址是数字的 IP 地址，例如`61.135.169.125`就是百度的官网地址之一，如果每次访问百度都需要输入 IP 的话，估计到今天互联网都还没有走出鸿蒙阶段。

<!-- more -->

在网络发展历史上，最开始确实就是直接使用 IP 地址来访问远程主机的。早期联网的每台计算机都是采用主机文件（即我们俗称的 hosts 文件）来进行地址配置和解析的，后来联网机器越来越多，主机文件的更新和同步就成了很大的问题。于是，1983 年保罗·莫卡派乔斯发明了域名解析服务和域名系统，在 1985 年 1 月 1 日，世界上第一个域名 [nordu.net](http://nordu.net/) 才被注册成功。

域名比 IP 地址更容易记忆，本质上只是为数字化的互联网资源提供了易于记忆的别名，就像在北京提起「故宫博物院」就都知道指的是「东城区景山前街 4 号」的那个大院子一样。如果把 IP 地址看成电话号码，那域名系统就是通讯录。我们在通讯录里保存了朋友和家人的信息，每次通过名字找到某人打电话的时候，通讯录就会查出与之关联的电话号码，然后拨号过去。我们可能记不下多少完整的电话号码，但是联系人的名字却是一定记得的。

既然「域名」只是一个别名，单凭这一个名字我们并不能访问到正确的地址，只有能将域名解析成实际的网络地址，网络访问才能成功。这种解析工作由专门的「域名系统」（Domain Name System，简称 DNS）完成，DNS 也是互联网的核心基础服务之一。

## 域名解析是怎么完成的

DNS 解析的过程是什么样子的呢？在开始这个问题之前，我们先看一看域名的层次结构。

### 域名的层级结构
在讨论域名的时候，我们经常听到有人说「顶级域名」、「一级域名」、「二级域名」等概念，域名级别究竟是怎么划分的呢？

* 根域名。还是以百度为例，通过一些域名解析工具，我们可以看到百度官网域名显示为 `www.baidu.com.`，细心的人会注意到，这里最后有一个 `.`，这不是 bug，而是所有域名的尾部都有一个根域名。`www.baidu.com` 真正的域名是 `www.baidu.com.root`，简写为`www.baidu.com`.，又因为根域名 `.root`对于所有域名都是一样的，所以平时是省略的，最终就变成了我们常见的样子。
* 根域名的下一级叫做顶级域名（top-level domain，缩写为 TLD），也叫做一级域名，常见的如 `.com/`、`.net/`、`.org/`、`.cn/` 等等，他们就是顶级域名。
* 再下一级叫做二级域名（second-level domain，缩写为 SLD），比如 `baidu.com`。这是我们能够购买和注册的最高级域名。
次级域名之下，就是主机名（host），也可以称为三级域名，比如 `www.baidu.com`，由此往下，基本上 N 级域名就是在 N-1 级域名前追加一级。

总结一下，常见的域名层级结构如下：
```
主机名.次级域名.顶级域名.根域名
www.baidu.com.root
```

一般来说我们购买一个域名就是购买一个二级域名（SLD）的管理权（如 0x2beace.com），有了这个管理权我们就可以随意设置三级、四级域名了。

### 域名解析的过程
与域名的分级结构对应，DNS 系统也是一个树状结构，不同级别的域名由不同的域名服务器来解析，整个过程是一个「层级式」的。

层级式域名解析体系的第一层就是根域名服务器，全世界 IPv4 根域名服务器只有 13 台（名字分别为 A 至 M），1 个为主根服务器在美国，其余 12 个均为辅根服务器，它们负责管理世界各国的域名信息。在根服务器下面是顶级域名服务器，即相关国家域名管理机构的数据库，如中国互联网络信息中心（CNNIC）。然后是再下一级的权威域名服务器和 ISP 的缓存服务器。

一个域名必须首先经过根数据库的解析后，才能转到顶级域名服务器进行解析，这一点与生活中问路的情形有几分相似。

假设北京市设立了一个专门的「道路咨询局」，里面设置了局长、部长、处长、科员好几个级别的公务员，不同的部门、科室、人员负责解答不同区域的道路问题。这里的人都有一个共同特点，信奉「好记性不如烂笔头」的哲理，喜欢将自己了解到的信息记录到笔记本上。但是有一点遗憾的是，他们写字用的墨水只有一种，叫「魔术墨水」，初写字迹浓厚，之后会慢慢变淡，1 小时之后则会完全消失。道路咨询局门口还有一个门卫大爷，所有的人要问路都需要通过他来传达和回复，市民并不能进入办公楼。

如果市民 A 先生来找门卫大爷询问「北海公园」的地址，门卫大爷会先看一下自己的笔记本，找找看之前有没有人问过北海公园，如果没有，他就会拨打内线去找局长求助。局长说北海是西城区，你去问负责西城区道路信息的赵部长吧。门卫大爷又去问赵部长，赵部长查了一下，说这个地址你去问负责核心区的钱处长吧。门卫大爷又给钱处长打过去电话，钱处长说这个地址我也不掌握啊，你去问一下负责景山片区的科员小孙吧。门卫大爷从小孙那里终于知道了北海公园地址，他赶紧记到自己的小本本上，然后把结果告诉了市民 A 先生。接下来一小时内，如果还有市民 B 先生再来问北海公园的话，门卫大爷就直接用笔记本上记载的结果回复了。当然，如果市民 C 女士过来问别的地址的话，门卫大爷就要把处理 A 先生问询的流程再走一遍了。

### 分级查询的实例
现在我们来看一个实际的例子。如果我们在浏览器中输入`https://news.qq.com`，那浏览器会从接收到的 URL 中抽取出域名字段（news.qq.com），然后将它传给 DNS 客户端（操作系统提供）来解析。

首先我们说明一下本机 DNS 配置（就是 `/etc/resolv.conf` 文件，里面指定了本地 DNS 服务器的地址，Windows 系统可能会有所不同）：

```shell
$ cat /etc/resolv.conf 
nameserver 202.106.0.20
nameserver 202.106.196.115
```

然后我们用 dig 这个工具查看一下 news.qq.com 的解析结果（其中中文部分是解释说明）：
```
$ dig news.qq.com

; <<>> DiG 9.10.6 <<>> news.qq.com
这是 dig 程序的版本号与要查询的域名

;; global options: +cmd
;; Got answer:
以下是要获取的内容。

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47559
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0
这个是返回应答的头部信息：
1. opcode：操作码，QUERY 代表查询操作；
2. status: 状态，NOERROR 代表没有错误;
3. id：编号，在 DNS 协议中通过编号匹配返回和查询；
4. flags: 标志，含义如下:
   - qr：query，查询标志，代表是查询操作
   - rd：recursion desired，代表希望进行递归查询操作;
   - ra：recursive available，代表查询的服务器支持递归查询操作;
5. QUERY 查询数，与下面 QUESTION SECTION 的记录数一一对应；
6. ANSWER 结果数，与下面的 ANSWER SECTION 的记录数一一对应；
7. AUTHORITY 权威回复数，如果查询结果由管理域名的域名服务器而不是缓存服务器提供的，则称为权威回复。
             0 表示所有结果都不是权威回复；
8. ADDITIONAL 额外记录数；

;; QUESTION SECTION:
;news.qq.com.			IN	A
查询部分,从左到右部分意义如下:
1、要查询的域名；
2、要查询信息的类别，IN 代表类别为 IP 协议，即 Internet。
3、查询的记录类型，A 记录(Address)代表要查询 IPv4 地址。

;; ANSWER SECTION:
news.qq.com.		136	IN	CNAME	https.qq.com.
https.qq.com.		476	IN	A	125.39.52.26
回应部分，从左到右各部分意义：
1、对应的域名
2、TTL，time to live，缓存时间，单位秒，代表缓存域名服务器可以在缓存中保存的期限。
3、查询信息的类别
4、查询的记录类型，CNAME 表示别名记录，A 记录(Address)代表 IPv4 地址。
5、域名对应的 ip 地址。

;; Query time: 56 msec
;; SERVER: 202.106.0.20#53(202.106.0.20)
查询使用的服务器地址和端口,其实就是本地 DNS 域名服务器
;; WHEN: Thu Jul 11 15:59:37 CST 2019
;; MSG SIZE  rcvd: 65
查询的时间与回应的大小，收到 65 字节的应答数据。
```

从这个应答可以看到，我们得到的结果不是权威回复，只是本地 DNS 服务器从缓存中给了应答。

接下来我们在 dig 命令中增加一个参数 `+trace`，看看完整的分级查询过程：
```
$ dig +trace news.qq.com

; <<>> DiG 9.10.6 <<>> +trace news.qq.com
;; global options: +cmd
.			432944	IN	NS	g.root-servers.net.
.			432944	IN	NS	k.root-servers.net.
.			432944	IN	NS	b.root-servers.net.
.			432944	IN	NS	h.root-servers.net.
.			432944	IN	NS	i.root-servers.net.
.			432944	IN	NS	f.root-servers.net.
.			432944	IN	NS	d.root-servers.net.
.			432944	IN	NS	e.root-servers.net.
.			432944	IN	NS	j.root-servers.net.
.			432944	IN	NS	l.root-servers.net.
.			432944	IN	NS	c.root-servers.net.
.			432944	IN	NS	m.root-servers.net.
.			432944	IN	NS	a.root-servers.net.
;; Received 228 bytes from 202.106.0.20#53(202.106.0.20) in 45 ms
这些就是神秘的根域名服务器，由本地 DNS 服务器返回了所有根域名服务器地址。

com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
;; Received 1171 bytes from 192.36.148.17#53(i.root-servers.net) in 57 ms
这里显示的是 .com 域名的 13 条 NS 记录，本地 DNS 服务器向这些顶级域名服务器发出查询请求，
询问 qq.com 的 NS 记录。

qq.com.			172800	IN	NS	ns1.qq.com.
qq.com.			172800	IN	NS	ns2.qq.com.
qq.com.			172800	IN	NS	ns3.qq.com.
qq.com.			172800	IN	NS	ns4.qq.com.
;; Received 805 bytes from 192.48.79.30#53(j.gtld-servers.net) in 331 ms
这里显示的是 qq.com 的 4 条 NS 记录，由 j.gtld-servers.net 这台服务器最先返回。
然后本地 DNS 服务器向这四台服务器查询下一级域名 news.qq.com 的 NS 记录。

news.qq.com.		86400	IN	NS	ns-cnc1.qq.com.
news.qq.com.		86400	IN	NS	ns-cnc2.qq.com.
;; Received 180 bytes from 58.144.154.100#53(ns4.qq.com) in 37 ms
这里显示的是 news.qq.com 的 NS 记录，它们是由上面的 ns4.qq.com 域名服务器返回的。
然后本地 DNS 服务器向这两台机器查询 news.qq.com 的主机名。

news.qq.com.		600	IN	CNAME	https.qq.com.
https.qq.com.		600	IN	A	125.39.52.26
;; Received 76 bytes from 223.167.83.104#53(ns-cnc2.qq.com) in 29 ms
这是上面的 ns-cnc2.qq.com 返回的最终查询结果：
news.qq.com 是 https.qq.com 的别名，而 https.qq.com 的 A 记录地址是 125.39.52.26
```
实际的流程里面，本地 DNS 服务器相当于门卫大爷，根域名服务器相当于局长同志，其余以此类推。客户端与本地 DNS 服务器之间的查询叫递归查询，本地 DNS 服务器与其他域名服务器之间的查询就叫迭代查询。

## 域名记录的类型

域名服务器之所以能知道域名与 IP 地址的映射信息，是因为我们在域名服务商那里提交了域名记录。购买了一个域名之后，我们需要在域名服务商那里设置域名解析的记录，域名服务商把这些记录推送到权威域名服务器，这样我们的域名才能正式生效。

在设置域名记录的时候，会遇到「A 记录」、「CNAME」 等不同类型，这正是前面做域名解析的时候我们碰到的结果。这些类型是什么意思，它们之间有什么区别呢？接下来我们看看常见的记录类型。

* **A 记录**。A (Address) 记录用来直接指定主机名（或域名）对应的 IP 地址。主机名就是域名前缀，常见有如下几种：
  * `www`：解析后的域名为 www.0x2beace.com，一般用于网站地址。
  * `@`：直接解析主域名。
  * `*`：泛解析，指将 *.yourdomain.com 解析到同一 IP。
* **CNAME 记录**。CNAME 的全称是 Canonical Name，通常称别名记录。如果需要将域名指向另一个域名，再由另一个域名提供 IP 地址，就需要添加 CNAME 记录。
* MX 记录。邮件交换记录，用于将以该域名为结尾的电子邮件指向对应的邮件服务器以进行处理。
* NS 记录。域名服务器记录，如果需要把子域名交给其他 DNS 服务器解析，就需要添加 NS 记录。
* AAAA 记录。用来指定主机名（或域名）对应的 IPv6 地址，不常用。
* TXT 记录。可以填写任何东西，长度限制 255。绝大多数的 TXT 记录是用来做 SPF 记录（反垃圾邮件），MX 记录的作用是给寄信者指明某个域名的邮件服务器有哪些。SPF 的作用跟 MX 相反，它向收信者表明，哪些邮件服务器是经过某个域名认可会发送邮件的。
* 显性 URL。从一个地址 301 重定向（也叫「永久性转移」）到另一个地址的时候，就需要添加显性 URL 记录。
* 隐性 URL。从一个地址 302 跳转（也叫「临时跳转」）到另一个地址，需要添加隐性 URL 记录。它类似于显性 URL，区别在于隐性 URL 不会改变地址栏中的域名。

在填写各种记录的时候，我们还会碰到一个特殊的设置项——TTL，生存时间（Time To Live）。

TTL表示解析记录在 DNS 服务器中的缓存时间，时间长度单位是秒，一般为3600秒。比如：在访问`news.qq.com` 时，如果在 DNS 服务器的缓存中没有该记录，就会向某个 NS 服务器发出请求，获得该记录后，该记录会在 DNS 服务器上保存TTL的时间长度，在TTL有效期内访问 `news.qq.com`，DNS 服务器会直接缓存中返回刚才的记录。

## 原文链接
* [域名背后那些事](https://leancloudblog.com/Domain-Name-Story-confirm/)

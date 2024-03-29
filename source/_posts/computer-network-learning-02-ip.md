---
title: 计算机网络学习——02 IP 篇
date: 2022-11-19 12:25:18
tags: ["计算机网络"]
categories: ["计算机网络"]
---

计算机网络学习——02 IP 篇

<!-- more -->

## IP 基本认识
IP 在 TCP/IP 参考模型中处于第三层，也就是网络层。

网络层的主要作用是：实现主机与主机之间的通信，也叫点对点（end to end）通信。

IP 的作用是在复杂的网络环境中，**将数据包发送给最终目的主机**。

### IP 与 MAC 的作用
IP 的作用是主机之间进行通信的，而 MAC 的作用是实现『直连』的两个设备之间通信，而 IP 则负责在「没有直连」的两个网络之间进行通信传输。

举个栗子。

小林要去一个很远的地方旅行，制定了一个行程表，其间需先后乘坐⻜机、地铁、公交⻋才能抵达目的地，为此小林需要买⻜机票，地铁票等。

⻜机票和地铁票都是去往特定的地点的，每张票只能够在某一限定区间内移动，此处的「区间内」就如同通信网络中数据链路。

**在区间内移动相当于数据链路层，充当区间内两个节点传输的功能**，区间内的出发点好比源 MAC 地址，目标地点 好比目的 MAC 地址。

整个旅游行程表就相当于网络层，充当远程定位的功能，行程的开始好比源 IP，行程的终点好比目的 IP 地址。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221009154601.png)

如果小林只有行程表而没有⻋票，就无法搭乘交通工具到达目的地。相反，如果除了⻋票而没有行程表，恐怕也很难到达目的地。因为小林不知道该坐什么⻋，也不知道该在哪里换乘。

因此，只有两者兼备，既有某个区间的⻋票又有整个旅行的行程表，才能保证到达目的地。与此类似，计算机网络 中也需要「数据链路层」和「网络层」这个分层才能实现向最终目标地址的通信。

还有􏰀要一点，旅行途中我们虽然不断变化了交通工具，但是旅行行程的起始地址和目的地址始终都没变。其实， 在网络中数据包传输中也是如此，**源IP地址和目标IP地址在传输过程中是不会变化的，只有源 MAC 地址和目标 MAC 一直在变化**。

## IP 地址基础知识

在 TCP/IP 网络通信时，为了保证能正常通信，每个设备都需要配置正确的 IP 地址，否则无法实现正常的通信。

IP 地址(IPv4 地址)由 32 位正整数来表示，IP 地址在计算机是以二进制的方式处理的。

而人类为了方便记忆采用了点分十进制的标记方式，也就是将 32 位 IP 地址以每 8 位为组，共分为 4 组，每组以 「 . 」隔开，再将每组转换成十进制。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220929204536.png)

### IP 地址的分类
#### 1. 网络地址

IP地址由网络号（包括子网号）和主机号组成，网络地址的主机号为全0，网络地址代表着整个网络。

#### 2. 广播地址

广播地址通常称为直接广播地址，是为了区分受限广播地址。

广播地址与网络地址的主机号正好相反，广播地址中，主机号为全1。当向某个网络的广播地址发送消息时，该网络内的所有主机都能收到该广播消息。

#### 3. 组播地址
D类地址就是组播地址。

先回忆下A，B，C，D类地址吧：
1. A类地址以0开头，第一个字节作为网络号，地址范围为：0.0.0.0~127.255.255.255；(modified @2016.05.31)
2. B类地址以10开头，前两个字节作为网络号，地址范围是：128.0.0.0~191.255.255.255;
3. C类地址以110开头，前三个字节作为网络号，地址范围是：192.0.0.0~223.255.255.255。
4. D类地址以1110开头，地址范围是224.0.0.0~239.255.255.255，D类地址作为组播地址（一对多的通信）；
5. E类地址以1111开头，地址范围是240.0.0.0~255.255.255.255，E类地址为保留地址，供以后使用。

注：只有A、B、C 有网络号和主机号之分，D类地址和E类地址没有划分网络号和主机号。

#### 4. 255.255.255.255
该IP地址指的是受限的广播地址。受限广播地址与一般广播地址（直接广播地址）的区别在于，受限广播地址只能用于本地网络，路由器不会转发以受限广播地址为目的地址的分组；一般广播地址既可在本地广播，也可跨网段广播。

例如：主机192.168.1.1/30上的直接广播数据包后，另外一个网段192.168.1.5/30也能收到该数据报；若发送受限广播数据报，则不能收到。

注：一般的广播地址（直接广播地址）能够通过某些路由器（当然不是所有的路由器），而受限的广播地址不能通过路由器。

#### 5. 0.0.0.0
常用于寻找自己的IP地址，例如在我们的RARP，BOOTP和DHCP协议中，若某个未知IP地址的无盘机想要知道自己的IP地址，它就以255.255.255.255为目的地址，向本地范围（具体而言是被各个路由器屏蔽的范围内）的服务器发送IP请求分组。

#### 6. 回环地址
127.0.0.0/8被用作回环地址，回环地址表示本机的地址，常用于对本机的测试，用的最多的是127.0.0.1。

#### 7. A、B、C类私有地址
私有地址(private address)也叫专用地址，它们不会在全球使用，只具有本地意义：
1. A类私有地址：10.0.0.0/8，范围是：10.0.0.0~10.255.255.255
2. B类私有地址：172.16.0.0/12，范围是：172.16.0.0~172.31.255.255
3. C类私有地址：192.168.0.0/16，范围是：192.168.0.0~192.168.255.255

## 参考链接
* [图解网络——小林coding](https://xiaolincoding.com/network/)
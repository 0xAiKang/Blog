---
title: 《程序是怎样跑起来的》读书笔记
date: 2021-01-21 22:01:32
tags: ["读书笔记"]
categories: ["读书笔记"]
---

《程序是怎样跑起来的》读书笔记

<!-- more -->

## CPU
CPU（计算机）能够直接识别和执行的只有机器语言。使用 C、Java 等语 言编写的程序，最后都会转化成机器语言。

CPU 和内存是由许多晶体管组成的电子部件，通常称为 IC (Integrated Circuit，集成电路)。

CPU 的内部由寄存器、控制器、运算器和时钟四个部分构成，各部分之间 由电流信号相互连通。
* 寄存器： 寄存器可用来暂存**指令**、**数据**等处理对象，可以将其看作是内存的一种。根据种类的不同，一个 CPU 内部会有20~100 个寄存器。
* 控制器：控制器负责把内存上的指令、数据等读入寄存器，并根据指令的执行结果来控制整个计算机。
* 运算器：运算器负责运算从内存读入寄存器的数据。 
* 时钟：时钟负责发出 CPU 开始计时的时钟信号。

时钟信号英文叫作 clock puzzle。Pentium 2 GHz 表示时钟信号的频率为 2 GHz(1 GHz = 10 亿次 / 秒)。也就是说，时钟信号的频率越高，CPU 的 运行速度越快。

通常我 们将汇编语言编写的程序转化成机器语言的过程称为 汇编;反之，机器语言程序转化成汇编语言程序的过程则称为 反汇编。

高级语言编写的程序 =》经过编译转换为机器语言=》CPU内部的寄存器来进行处理。

编译指的是使用高级编程语言编写的程序转换为机器语言的过程，其中，用于转换的程序被称为编译器。

对于程序员来说，CPU 是什么呢？CPU 是具有各种功能的寄存器的集合体，所以可以将寄存器理解成是CPU 的核心，主要承担着指令、数据的处理。

二进制转十进制的方式：即各位数的数值和位权相乘后再相加的数值。

位权的概念：39 = 3 * 10 + 9 * 1 其中 10 和 1 就是位权。
在十进制中，第 1 位(最右边的一位) 是 10 的 0 次幂 A(= 1)，第 2 位是 10 的 1 次幂(= 10)，第 3 位是 10 的 2 次幂(= 100)。

在二进制中，第 1 位是 2 的 0 次幂 (= 1)，第2位是2的1次幂(= 2)，这就是位权。

无论程序中使用的是多少进制，计算机最终都会转换为二进制来处理。

## 二进制
二进制的运算方式是：
对于十进制，进行加法运算时逢十进一，进行减法运算时借一当十；
对于二进制，进行加法运算时逢二进一，进行减法运算时借一当二。

二进制数中表示负数值时，一般会把最高位作为符号来使用，因此我们把这个最高位称为符号位。 符号位是 0 时表示正数 ，符号位是 1 时表示负数。

将二进制数的值取反后加 1 的结果，和原来的值相加，结果为0 。
实际上就是1 + (-1) = 0

### 计算机进行小数运算

> 为什么将0.1 累加一百次无法得到 10？

这是因为计算机无法准确用二进制表示 0.1，

十进制的0.1 转换成二进制后，会变成`0.00011001100...`(1100 循环)这样的 循环小数，这和无法用十进制准确表示 1/3 一样的道理。

因此，在 遇到循环小数时，计算机就会根据变量数据类型所对应的长度将数值 从中间截断或者四舍五入。

小 数 点 后 4 位 用 二 进 制 数 表 示 时 的 数 值 范 围 为 `0.0000~0.1111`。因此，这里只能表示 0.5、0.25、0.125、0.0625 这四个 二进制数小数点后面的位权组合而成(相加总和)的小数。

所以0.5 累加一百次可以到的 50，而0.1 累加一百次则会丢失精度。

#### 二进制和十进制
在实际的程序中，往往不会直接使用二进制来表示，因为太长了，一个二进制就需要八位来表示。

二进制数的 4 位，正好相当于十六进制数的 1 位。

## 内存
其实，从物理上来看，内存的构造非常简单。只要在程序上花一些心思，就可以将内存变换成各种各样的数据结构来使用。

内存实际上是一种名为内存 IC 的电子元件。

内存 IC 中有电源、地址信号、数据信号、控制信号等用于输入输出的大量引脚(IC 的引脚)，通过为其指定地址(address)，来进行数据的读写。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210121215618.png)

那么，这个内存IC 中能存放多少数据呢？

1. 数据信号引脚有 D0~D7共八个，表示一次可以输入输出 8 位(= 1 字节)的数据。
2. 地址信
号引脚有 A0~A9 共十个，表示可以指定 `0000000000~1111111111` 共1024 个地址。
3. 而地址是用来表示数据的存储场所，因此我们可以得出这 A个内存 IC 中可以存储 1024 个 1 字节的数据。因为 1024 = 1K，所以改内存IC 的容量是1KB。

### 指针
指针也是一种变量，它所表示的不是数据的值，而是存储着数据的内存的地址。

通过使用指针，就可以对任意指定地址的数据进行读写。

数组的定义中所指定的数据类型，也表示一次能够读写的内存大小。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210121215650.png)

高级编程语言的数组则完全省略了这些概念，直接定义一个数组，就可以放入任意类型的数据（int、float、double、string、object等）。

指针的概念也是类似，指针的数据类型表示一次可以读写的长度。

### 栈、队列及环形缓冲区
栈的原意是“干草堆积如山”。干草堆积成山后，最后堆的干草会 被最先抽取出来（后进先出）。

而队列则是完全相反的一种数据结构，先进先出。

## 内存和磁盘

### 不读入内存就无法运行
计算机中主要的存储部件是内存和磁盘。磁盘中存储的程序，必须要加载到内存后才能运行。

> 为什么程序一定要在内存中运行？

这是因为，这是因为负责解析和运行程序内容的CPU，需要通过内部程序计数器来指定内存地址，然后才能读出地址。

即使CPU 可以直接读出并运行磁盘中保存的程序，由于磁盘读取速度慢，程序的运行速度还是会降低。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210121215717.png)

本文中的所有图片均来自《程序是如何跑起来的》。

### 磁盘缓存加速了磁盘访问速度
磁盘缓存指的是把从磁盘中读取的数据存储到内存中的方式。

磁盘访问提高访问速度的机制：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210121215737.png)

### 虚拟内存把磁盘作为部分内存来使用
虚拟内存是把磁盘作为假象的内存来使用。这与磁盘缓存是假象的磁盘（实际是内存）是相对的，虚拟内存是假象的内存（实际是磁盘）。

## 亲自尝试压缩数据
文件是将数据存储在磁盘等存储媒介中的一种形式，程序文件中存储数据的单位是字节。

我们把能还原到压缩前状态的压缩称为 **可逆压缩**，无法还原到压 缩前状态的压缩称为 **非可逆压缩**。

## 从源文件到可执行文件

在程序运行时，用来动态申请分配数据和对象的内存区域形式称为**堆**。

源代码编译 =》本地代码（机器代码）=》dump（每个字节用 2 位十六进制数来表示的方式）

仅靠编译是无法得到可执行文件的，编译器编译仅仅只是得到了本地文件，为了得到可执行文件，还需要进行”链接“处理。

### 编译
在Windows 下，编译后生成的不是 EXE 文件，而是扩展名为`.obj`的目标文件，在Unix 下，编译后生成的也不是可执行文件，而是扩展名为`.o` 的目标文件。

这些文件无法直接运行，这是因为编译过程只是检查语法（函数、变量的声明）是否正确。

Mac 下编译`main.cpp` 文件：
```
$ gcc -c main.cpp
$ ls 
main.cpp  main.o
```

### 链接
找到所要用到函数所在的目标文件并结合，生成一个可执行文件的处理就是链接，运行连接的程序被称为链接器。

Mac 下链接`main.o` 文件：
```
$ gcc main.o -o main
$ main.cpp  main.o  main
```

两步可以合并成一步：
```
$ gcc main.cpp
$ ls 
main.cpp  a.out
```

总结：
* `main.cpp`：源代码文件
* `main.o`：源代码文件编译后生成的本地代码（机器语言）
* `main`：可执行文件
* `a.out`：可执行文件（默认名称）
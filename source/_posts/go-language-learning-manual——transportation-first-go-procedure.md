---
title: Go 语言学习笔记——运行第一个 Go 程序
date: 2022-10-21 22:41:54
tags: ["Go"]
categories: ["Go"]
---

时隔一年之久，再一次来学习 Go。

<!-- more -->

## 设计哲学
Go 语言的设计哲学：**简单、显式、组合、并发和面向工程**：
* 简单是指 Go 语言特性始终保持在少且足够的水平，不走语言特性融合的道路，但又不乏生产力。简单是 Go 生产力的源泉，也是 Go 对开发者的最大吸引力
* 显式是指任何代码行为都需开发者明确知晓，不存在因“暗箱操作”而导致可维护性降低和不安全的结果
* 组合是构建 Go 程序骨架的主要方式，它可以大幅降低程序元素间的耦合，提供程序的可扩展性和灵活性
* 并发是 Go 敏锐地把握了 CPU 向多核方向发展这一趋势的结果，可以让开发人员在多核时代更容易写出充分利用系统资源、支持性能随 CPU 核数增加而自然提升的应用程序
* 面向工程是 Go 语言在语言设计上的一个重大创新，它将语言要解决的问题域扩展到那些原本并不是由编程语言去解决的领域，从而覆盖了更多开发者在开发过程遇到的“痛点”，为开发者提供了更好的使用体验

## 安装
Go 从 2009 年开源并演化到今天，它的安装方法其实都已经很成熟了。

写下这篇笔记时，Go 的版本已经到了 1.18.1。

Windows 和 Mac 可以直接从官网[下载](https://golang.google.cn/dl/) 最新的安装包，在图形界面的引导下，一路“下一步”，即可安装成功。

Linux 则通过命令行的方式进行安装：
```bash
$ wget -c https://go.dev/dl/go1.19.2.linux-amd64.tar.gz \
tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
```
解压成功，即可在 `/usr/local` 下面看到名为 go 的安装目录。

添加环境变量：
```bash
$ export PATH=$PATH:/usr/local/go/bin && source ~/.profile
```

查看是否安装成功：
```bash
$ go version

go version go1.19.2 darwin/amd64
```

## 配置 Go
其实 Go 在安装后是开箱即用的，无需做任何配置就能使用。

但是因为众所周知的原因，一般会修改 `GOPROXY` 环境变量：
```bash
$ go env -w GOPROXY=https://goproxy.cn,direct

# 备用goproxy服务 https://mirrors.aliyun.com/goproxy/
```

顺便看一下其他的一些常用配置项：

| 名称|作用|值|
| ------- | ------- | ------- |
| GOARCH | 用于指示编译器生成代码（针对平台 CPU 架构）| 主要值是 AMD64、Arm 等，默认值是本机的 CPU 架构 |
| GOOS | 用于指示 Go 编译器生成代码（针对平台的操作系统 | 主要值是 Linux、Darwin、Windows 等，默认值是本机的操作系统 |
| GO111MODULE |  它的值决定了当前使用的构建模式是传统的 GOPATH 模式还是新引入的 Go Module 模式| 在 Go1.16 版本 Go Module 构建模式默认开启，该变量的值是 on |
| GOCACHE |  用于指示存储构建结果缓存的路径，这些缓存可能会被后续构建所使用 | 在不同的操作系统上，GOCACHE 有不同的默认值，通过 go env GOMODCACHE 查看 |
| GOMODCACHE |  用于指示存放 Go Module 的路径 | 在不同的操作系统上，GOCACHE 有不同的默认值，通过 go env GOMODCACHE 查看 |
| GOPROXY |  用来配置 Go Module proxy 服务 | 默认值是 `https://proxy.golang.org.direct` |
| GOPATH | 在传统的 GOPATH 构建模式下，用于指示 Go 包搜索路径的环境变量，在 Go module 机制启用之前是 Go 核心配置项，Go 1.8 版本之前需要手动配置，Go 1.8 版本之后引入了默认的GOPATH（$HOME/go） | |
| GOROOT | 指示 GO 安装路径，GO 1.10 版本引入了默认的 GOROOT，开发者无需显式设置，Go 程序会自动根据自己所在的路径推导出 GOROOT 的路径   |

## hello world
首先，需要创建一个 `main.go` 的源文件。

这里需要注意一下 Go 的命名规则：Go 源文件总是用**全小写字母形式的短小单词命名**，并且 `.go` 扩展名结尾。

如果要在源文件的名字中使用多个单词，通常直接是将多个单词连接起来作为源文件名，而不是使用其他分割符。\
比如下划线，通常会使用 `helloworld.go` 作为文件名，而不是 `hello_world.go`。

这是因为下划线这种分割符，在 Go 源文件命名中有特殊作用。

```go
// main.go

package main

import "fmt"

func main(){
  fmt.Println("hello world")
}
```

写完之后，就可以编译运行第一个 Go 程序了：
```bash
$ go build main.go   # 编译

$ ./main             # 运行
hello, world
```

上面这个简单的 Go 程序，由三个很重要的部分组成：
* `package main`：定义了 Go 中的一个包 package
* `import fmt`：声明导入标准库 fmt 目录下的包
* `func main(){}`：定义入口函数

下面一一说明。

### package
包（package）是 Go 语言的基本组成单元，通常使用单个的小写单词命名，一个 Go 程序本质上就是一组包的集合。

所有 Go 代码都有属于自己的 package（很像 PHP 的命名空间的概念），在这里的“helloworld”示例的所有代码都在一个名为 main 的包中。main 包在 Go 中是一个特殊的包，**整个 Go 程序中仅允许存在一个名为 main 的包**。

### import
`import` 的作用是导入标准库或者第三方包，在这里的作用是导入标准库 fmt 目录下的包。

在上面的示例中，有两处都使用了 `fmt` 这个字面值，但是其含义是不一样的。
* `import fmt` 一行中“fmt”代表的是包的导入路径（Import），它表示的是**标准库下的 fmt 目录**
* `fmt.Println` 函数调用一行中的“fmt”代表的则是**包名**

main 函数体中之所以可以调用 fmt 包的 Println 函数，还有最后一个原因，那就是 Println 函数名的首字母是大写的。

在 Go 语言中，**只有首字母为大写的标识符才是导出的（Exported），才能对包外的代码可见**；如果首字母是小写的，那么就说明这个标识符**仅限于在声明它的包内可见**。

### main
main 包中的主要代码是一个名为 main 的函数：

```go
func main() {
    fmt.Println("hello, world")
}
```

这里的 main 函数会比较特殊：**当运行一个可执行的 Go 程序的时，所有的代码都会从这个入口函数开始运行**。

---

Go 要求所有的函数体都要被花括号包裹起来，用来标记函数体。按照惯例，推荐把左花括号与函数声明置于同一行并以空格分隔。\
Go 语言内置了一套 Go 社区约定俗称的代码风格，并随安装包提供了一个名为 Gofmt 的工具，这个工具可以帮助将代码自动格式化为约定的风格。

---
通过观察输出，可以发现，传入的字符串就是执行程序后在终端的标准输出上看到的字符串。

这种“所见即所得”得益于 Go 源码文件本身采用的是 Unicode 字符集，而且用的是 UTF-8 标准的字符编码方式，这与编译后的程序所运行的环境所使用的字符集和字符编码方式是一致的。

---
整个示例程序源码中，都没有使用过分号来标识语句的结束，这是因为，大多数分号都是可选的，常常被省略，不过在源码编译时，Go 编译器会自动插入这些被省略的分号。

所以加上分号也是完全合法的，只不过 gofmt 在按约定格式化代码时，会自动删除这些分号。

## 总结
通过 `hello world` 示例程序，了解了 Go 的基本源码结构，以下是重点：
* 包是 Go 语言的基本组成单元。一个 Go 程序就是一组包的集合，所有 Go 代码都位于包中
* Go 源码可以导入其他 Go 包，并使用其中的导出语法元素，包括类型、变量、函数、方法等
* 整个 Go 程序中仅允许存在一个名为 main 的包只能由一个 main 包; main 函数是整个 Go 应用的入口函数

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
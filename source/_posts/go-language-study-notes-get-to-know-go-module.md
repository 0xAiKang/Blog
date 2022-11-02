---
title: Go 语言学习笔记——认识Go Module
date: 2022-10-22 21:30:54
tags: ["Go"]
categories: ["Go"]
---

Go module 构建模式是在 Go 1.11 版本正式引入的，为的是**彻底解决 Go 项目复杂版本依赖的问题**，在 Go 1.16 版本中，Go module 已经成为了 Go 默认的包依赖管理机制和 Go 源码构建机制。

<!-- more -->

Go Module 的核心是一个名为 `go.mod` 的文件，在这个文件中存储了这个 module 对第三方依赖的全部信息。

## go module
首先来创建一个 `hellomodule.go` 的源文件。

```go
// hellomodule.go

package main

import (
	"github.com/valyala/fasthttp"
	"go.uber.org/zap"
)

var logger *zap.Logger

func init() {
	logger, _ = zap.NewProduction()
}
func fastHTTPHandler(ctx *fasthttp.RequestCtx) {
	logger.Info("hello, go module", zap.ByteString("uri", ctx.RequestURI()))
}
func main() {
	fasthttp.ListenAndServe(":8081", fastHTTPHandler)
}
```
先不用在意每行代码的意思，只需要知道在这个示例中，通过 import 引入了两个第三方依赖库。

接下来，通过下面命令为 “hellomodule” 这个示例程序添加 `go.mod` 文件：

```bash
$ go mod init hellomodule
go: creating new go.mod: module hellomodule
go: to add module requirements and sums:
  go mod tidy
```

`go mod init` 命令的执行结果是在当前目录下生成了一个 `go.mod` 文件，查看一下：
```bash
$ cat go.mod

module hellomodule

go 1.18
```

其实，一个 module 就是一个包的集合，这些包和 module 一起打版本、发布和分发。\
`go.mod` 所在的目录被我们称为它声明的 module 的根目录。

有了 `go.mod` 之后，还不能立马构建 “hellomodule” ，因为需要添加源码依赖，也就是代码中用到的 fasthttp 和 zap 这两个第三方包。

使用以下命令自动添加：
```bash
$ go mod tidy
```

再次查看 `go.mod` 文件，就会发现多了很多内容：
```go
$ cat go.mod

module hellomodule

go 1.18

require (
        github.com/valyala/fasthttp v1.41.0
        go.uber.org/zap v1.23.0
)

require (
        github.com/andybalholm/brotli v1.0.4 // indirect
        github.com/klauspost/compress v1.15.9 // indirect
        github.com/valyala/bytebufferpool v1.0.0 // indirect
        go.uber.org/atomic v1.7.0 // indirect
        go.uber.org/multierr v1.6.0 // indirect
)
```

还会发现本地多了一个 `go.sum` 的文件，它的作用是记录项目的直接依赖和间接依赖包的相关版本的 hash 值，用来校验本地包的真实性。\
在构建的时候，如果本地依赖包的 hash 值与 `go.sum` 文件中记录的不一致，就会被拒绝构建。

现在就可以进行编译了：
```bash
$ go build hellomodule.go
$ ls
go.mod    go.sum    hellomodule*    hellomodule.go

$ ./hellomodule
```

启动服务，然后访问 `localhost:8081`，可以看到控制台输出以下内容，即表示服务正常
```json
{"level":"info","ts":1667202109.444561,"caller":"hellomodule/hellomodule.go:14","msg":"hello, go module","uri":"/"}
```

## 总结
* Go 源码需要先编译，再分发和运行
* 如果是单 Go 源文件的情况，可以直接使用 go build 命令 + Go 源文件名的方式编译
* 对于复杂的 Go 项目，需要在 Go Module 的帮助下完成项目的构建
* Go Module 已经是 go 官方标准包依赖管理和构建模式了，gopath 模式了解即可
* `go mod init` 命令为项目创建一个 Go Module
* `go mod tidy` 命令自动添加第三方依赖

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
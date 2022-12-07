---
title: Go 语言学习笔记——Go 标准命令学习
date: 2022-12-01 17:56:58
tags: ["Go"]
categories: ["Go"]
---

Go 语言标准命令详解。

<!-- more -->

## go env
`go env` 是非常常用的命令，用于查看当前的 go 环境信息。

后面跟上环境变量名称，则可以输出对应环境变量的值：
```bash
$ go env GOROOT
/usr/local/go
```

> 如果要修改某一个环境变量的值，那么该如何修改呢？
> 

不同的操作系统，修改的位置不同：
* Mac 和 Linux 是 `~/.bashrc` 或 `~/.bash_profile` 文件
* 如果使用了Zsh 那么就是 `~/.zshrc` 文件
* Windows 则需要去系统设置里面的环境变量中修改

```bash
# 更新环境变量的值
export GOROOT=/usr/local/go
# 加入环境变量中
export PATH=$PATH:$GOROOT/bin
```

添加以上内容到环境配置文件中，并保存，执行`source ~/.zshrc`命令刷新环境变量即可。

## go build

`go build` 也是非常常用的命令，用于对 Go 程序进行编译，命令格式如下：
```bash
$ go build [-o output] [-i] [build flags] [packages]
```

有两种情况：
1. 对于普通包（ 非 main 包）而言，只会执行编译检查, 不会产生任何文件
2. 对于 main 包除了进行编译检查外，还会在当前目录下生成一个可执行文件

如果当前目录下有多个文件，却只想编译某个文件，可以在命令后面指定文件名称：
```bash
$ go build main.go
```

Go提供了编译链工具，可以让我们在任何一个开发平台上，编译出其他平台的可执行文件。

默认情况下，都是根据当前的机器生成的可执行文件，如果需要在当前环境编译出其他操作系统的可执行文件，那么使用下面的命令：
```bash
$ GOOS=linux GOARCH=amd64 go build bash
```
GOOS 表示的是操作系统的名称，GOARCH 表示的是目标处理器的架构。

## go run

`go build` 命令是对程序进行编译，`go run` 则是对程序进行运行，相当于是把编译和执行二进制文件这两步，合并成了一步：
```bash
$ go run hello.go
hello world
```

## go install

`go install` 和 `go build` 类似，不过它可以在编译后，会把生成的可执行文件或者库安装到对应的目录下，以供使用。

它的用法和 `go build` 差不多，如果不指定一个包名，就使用当前目录。安装的目录都是约定好的，如果生成的是可执行文件，那么安装在 `$GOPATH/bin` 目录下；如果是可引用的库，那么安装在 `$GOPATH/pkg` 目录下。

有时候一些第三方的依赖会提供一些命令行工具，这个时候就会用到 `go install` 命令进行安装了，例如安装 [Air](https://github.com/cosmtrek/air)：
```bash
$ GO111MODULE=on go install github.com/cosmtrek/air@latest
```

## go get
`go get` 命令也是十分常用的，用于下载更新指定的包以及依赖的包，并对它们进行编译和安装。

如果需要更新某个依赖包，则加上 `-u` 参数：
```bash
$ go get -u golang.org/x/sys
```

下载下来的依赖包，全部在 `$GOPATH` 目录下。

## go fmt
`go fmt` 用于统一代码风格，`go fmt`会自动格式化代码文件并保存，它本质上其实是调用的 `gofmt -l -w` 这个命令。

使用 GoLand 进行开发时，每次编辑完一个文件，都会自动进行 `gofmt` 格式化。

## go test
`go test` 命令用于Go 的单元测试，它也是接受一个包名作为参数，如果没有指定，使用当前目录。 `go test` 运行的单元测试必须符合 Go 的测试要求。

1. 写有单元测试的文件名，必须以_test.go结尾。
2. 测试文件要包含若干个测试函数。
3. 这些测试函数要以Test为前缀，还要接收一个*testing.T类型的参数。

```go
package test

import (
	"log"
	"strings"
	"testing"
)

func TestName(t *testing.T) {
	if "hello" == strings.ToLower("Hello") {
		log.Println("ok")
	}
}
```

这是一个单元测试，保存在 `default_test.go` 文件中。 如果要运行这个单元测试，在该文件目录下，执行 `go test default_test.go` 即可。
```bash
$ go test default_test.go 
ok      command-line-arguments  0.083s
```

## go vet
`go vet` 命令用于检查代码中常见的错误，其中包括：
1. Printf 这类的函数调用时，类型匹配了错误的参数。
2. 定义常用的方法时，方法签名错误。
3. 错误的结构标签。
4. 没有指定字段名的结构字面量。

```go
package main

import "fmt"

func main() {
	fmt.Printf("hello", "world")
}
```

这是一个很明显的错误例子，格式字符串中没有占位符，使用 `go vet` 命令就可以发现这个错误：

```bash
$ go vet hello.go          
# command-line-arguments
./hello.go:6:12: fmt.Printf call has arguments but no formatting directives
```

使用 GoLand 进行开发时，对于这种类型的错误，编译器会给出一个警告。

## go list

`go list` 命令的作用是列出指定的代码包的信息。

查看当前包所使用的所有第三方依赖：
```bash
$ go list -m all
```

`go list` 命令还有许多其他用法，通过 `go help list` 进行查看。

## 参考链接
* [标准命令详解](https://doc.yonyoucloud.com/doc/wiki/project/go-command-tutorial/0.0.html)
* [命令文档——Go](https://go-zh.org/cmd/go/)
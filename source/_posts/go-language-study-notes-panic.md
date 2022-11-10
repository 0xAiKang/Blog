---
title: Go 语言学习笔记——panic
date: 2022-10-28 23:16:51
tags: ["Go"]
categories: ["Go"]
---

Go 函数的健壮性设计包括很多方面，首先是最基本的“三不要”原则，简单来了解一下。

<!-- more -->

## 三不要原则

### 不要相信任何外部输入的参数

函数的使用者可能是任何人，这些人在使用函数之前可能都没有阅读过任何手册或文档，他们会向函数传入你意想不到的参数。因此，为了保证函数的健壮性，函数需要对所有输入的参数进行合法性的检查。

一旦发现问题，立即终止函数的执行，返回预设的错误值。

### 不要忽略任何一个错误

在函数实现中，通常会调用标准库或第三方包提供的函数或方法。对于这些调用，不能假定它一定会成功，一定要显式地检查这些调用返回的错误值。

一旦发现错误，要及时终止函数执行，防止错误继续传播。

### 不要假定异常不会发生

先要确定一个认知：**异常不是错误**。

错误是可预期的，也是经常会发生的，有对应的公开错误码和错误处理预案，但异常却是少见的、意料之外的。通常意义上的异常，指的是硬件异常、操作系统异常、语言运行时异常，还有更大可能是代码中潜在 bug 导致的异常，比如代码中出现了以 0 作为分母，或者是数组越界访问等情况。

虽然异常发生是“小众事件”，但是不能假定异常就不会发生。

## panic

不同编程语言表示异常（Exception）这个概念的语法都不相同，在 Go 语言中，异常这个概念由 panic 表示。

panic 指的是 Go 程序在运行时出现的一个异常情况。如果异常出现了，但没有被捕获并恢复，Go 程序的执行就会被终止，即便出现异常的位置不在主 Goroutine 中也会这样。

在 Go 中，panic 主要有两类来源，一类是来自 **Go 运行时**，另一类则是 Go 开发人员通过 **panic 函数主动触发的**。\
无论是哪种，一旦 panic 被触发，后续 Go 程序的执行过程都是一样的，这个过程被 Go 语言称为 **panicking**。

下面用一个例子来直观感受一下 panicking 这个过程：
```go
func foo() {
	println("call foo")
	bar()
	println("exit foo")
}

func bar() {
	println("call bar")
	panic("panic occurs in bar")
	zoo()
	println("exit bar")
}

func zoo() {
	println("call zoo")
	println("exit zoo")
}

func main() {
	println("call main")
	foo()
	println("exit main")
}
```

在上面的示例中，从 main 函数开始，函数的调用次序依次为 `main` -> `foo` -> `bar` -> `zoo`。在 bar 函数中，调用 panic 函数手动触发了 panic。

最终程序的输出结果是：
```
call main
call foo
call bar
panic: panic occurs in bar
```

下面用一张图来解释程序的调用过程：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221106220207.png)

关键部分有两处：
1. 在 bar 函数中，代码在执行下一个函数之前调用了 panic 函数触发了异常，所以 bar 函数的执行就此停住了，panicking 过程就此开始
2. 因为没有捕获 panic，所以 panic 会沿着函数调用栈一直向下走，从 foo/main 这些函数的视角来看，对 bar/foo 函数的调用，和对 panic 函数的调用是一样的，因为都没有捕捉 panic，所以调用完 panic 之后，自身的执行就此停止了，已经执行完成的函数会依次从栈顶弹出，最后main 函数也 exit 了

不过，Go 也提供了捕捉 panic 并恢复程序正常执行秩序的方法，我们可以通过 `recover` 函数来实现这一点。

用上面这个例子分析，在触发 panic 的 bar 函数中，对 panic 进行捕捉并恢复，直接来看恢复后，整个程序的执行情况是什么样的（除了 bar 函数调整了，其他函数均没有改变）：
```go
func bar() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Println("recover the panic:", e)
        }
    }()
    println("call bar")
    panic("panic occurs in bar")
    zoo()
    println("exit bar")
}
```

在更新版的 bar 函数中，通过 defer 匿名函数中调用 recover 函数对 panic 进行了捕获：
* 如果捕获到，panic 引发的 panicking 过程就会停止，并返回以 panic 的具体内容为错误上下文信息的错误值
* 如果没有 panic 发生，那么 recover 将返回 nil

执行更新后的程序，得到如下结果：
```
call main
call foo
call bar
recover the panic: panic occurs in bar
exit foo
exit main
```

调用过程如下图所示：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221106221146.png)

可以看到，此时 main 函数是正常执行完退出的，因为使用了 recover 顺利捕获到了 panic。

面对有如此行为特点的 panic，那么到底该如何使用呢？是不是在所有 Go 函数或方法中，都要用 defer 函数来捕捉和恢复 panic 呢？

## 如何应对 panic
其实不用，原因有两点：
1. 大量的 panic 会徒增开发人员实现函数时的心智负担
2. 很多函数非常简单，根本不会出现 panic 情况，增加 panic 捕获和恢复，反倒会增加函数的复杂性。同时，defer 函数会带来一定性能开销

下面提供三点经验，可以参考一下。

### 评估 panic 等级

首先，应该知道一个事实：**不同应用对异常引起的程序崩溃退出的忍受度是不一样的。**

比如，一个单次运行于控制台窗口中的命令行交互类程序（CLI），和一个常驻内存的后端 HTTP 服务器程序，前者即便因异常崩溃，对用户来说也仅仅是再重新运行一次而已。但后者一旦崩溃，就很可能导致整个网站停止服务。

所以针对各种应用对 panic 忍受度的差异，应该采取的 panic 的策略也是不同的。

像后端 HTTP 服务器程序这样的任务关键系统，就需要在特定的位置捕获并恢复 panic，以保证服务器整体的健壮度。

### 提示潜在 bug

当一些本不该发生的事情导致程序异常结束时，可以使用 panic 充当类似断言的作用。

在 json 包的 `encode.go` 中也有使用 panic 充当断言的例子：
```go
// $GOROOT/src/encoding/json/encode.go
func (w *reflectWithString) resolve() error {
    ... ...
    switch w.k.Kind() {
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        w.ks = strconv.FormatInt(w.k.Int(), 10)
        return nil
    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        w.ks = strconv.FormatUint(w.k.Uint(), 10)
        return nil
    }
    panic("unexpected map key type")
}
```
这段代码中，resolve 方法的最后一行代码就相当于一个“代码逻辑不会走到这里”的断言。一旦触发“断言”，这很可能就是一个潜在 bug。

去掉 panic 这行代码并不会对程序造成影响，但是如果存在的话，当问题出现时，就可以借助 panic 作为断言快速定位到问题所在。

### 不要混淆异常与错误
在 Go 中，通常会导入大量第三方包，而对于这些第三方包 API 中是否会引发panic，调用者是不知道的。

因此上层代码，也就是 API 调用者根本不会去逐一了解 API 是否会引发panic，也没有义务去处理引发的 panic。因此，在 Go 中，API 的提供者，一定不要将 panic 当作错误返回给 API 调用者。

## 总结
* 错误是 error，异常是 panic，两者是有本质区别的
* defer 要在panic 之前，才能执行
* recover 只能在defer 中调用才能生效
* defer 内部的recover 只能捕获当前协程的Panic，不能跨协程执行
* 无论在哪个 Goroutine 中发生未被恢复的 panic，整个程序都将崩溃退出

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
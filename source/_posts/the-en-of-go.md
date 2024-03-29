---
title: Go语言之禅
date: 2022-12-18 16:11:11
tags: ["Go"]
categories: ["Go"]
---

Go语言之禅

<!-- more -->

## Go 箴言
* 不要通过共享内存进行通信，通过通信共享内存
* 并发不是并行
* 管道用于协调；互斥量（锁）用于同步
* 接口越大，抽象就越弱
* 利用好零值
* 空接口 interface{} 没有任何类型约束
* Gofmt 的风格不是人们最喜欢的，但 gofmt 是每个人的最爱
* 允许一点点重复比引入一点点依赖更好
* 系统调用必须始终使用构建标记进行保护
* 必须始终使用构建标记保护 Cgo
* Cgo 不是 Go
* 使用标准库的 unsafe 包，不能保证能如期运行
* 清晰比聪明更好
* 反射永远不清晰
* 错误是值
* 不要只检查错误，还要优雅地处理它们
* 设计架构，命名组件，（文档）记录细节
* 文档是供用户使用的
* 不要（在生产环境）使用 panic()

## Go 语言之禅
* 每个 package 实现单一的目的
* 显式处理错误
* 尽早返回，而不是使用深嵌套
* 让调用者处理并发（带来的问题）
* 在启动一个 goroutine 时，需要知道何时它会停止
* 避免 package 级别的状态
* 简单很重要
* 编写测试以锁定 package API 的行为
* 如果你觉得慢，先编写 benchmark 来证明
* 适度是一种美德
* 可维护性

## 参考链接
* [The Zen of Go](https://the-zen-of-go.netlify.com/)
* [Go Proverbs](https://go-proverbs.github.io/)
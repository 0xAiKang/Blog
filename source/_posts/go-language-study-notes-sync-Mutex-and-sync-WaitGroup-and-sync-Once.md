---
title: Go 语言学习笔记——sync.WaitGroup和sync.Once
date: 2022-11-09 14:30:34
tags: ["Go"]
categories: ["Go"]
---

这篇笔记主要用来介绍 Go 的sync 包的 WaitGroup 类型，它也是并发编程中，经常被用到的一个同步工具。

<!-- more -->

## WaitGroup类型
sync 包的 WaitGroup 类型（以下简称WaitGroup类型），通常用来实现一对一或者一对多的 goroutine  协作流程。

WaitGroup类型，也是零值是可用的，因此不需要初始化，开箱即用。

可以把 WaitGroup 类型理解成是一个计数信号量，用来记录并维护运行的 goroutine。

WaitGroup类型拥有三个指针方法：
* Add：对 WaitGroup 类型值的增加或者减少计数器的值
* Done：等待 goroutine 执行完成通知 main goroutine
* Wait：阻塞当前的 goroutine，直到其所属值中的计数器归零

下面通过一段示例代码来加深印象：
```go
// 声明一个 WaitGroup 类型的变量
var wg sync.WaitGroup

func worker(id int) {
	fmt.Printf("Worker %d starting\n", id)

	time.Sleep(time.Second)
	fmt.Printf("Worker %d done\n", id)
}

func main() {
  // 显示 Add 方法，计数器增加 5
	wg.Add(5)
  
	for i := 1; i <= 5; i++ {
		// 创建中间变量，避免在每个协程闭包中重复利用相同的 i 值
		i := i
		go func() {
			// 通知 WaitGroup 此工作线程已执行完成
			defer wg.Done()
			worker(i)
		}()
	}
	// 阻塞，直到 WaitGroup 计数器恢复为 0； 即所有goroutine 都执行完了
	wg.Wait()
}
```

首先声明一个全局 WaitGroup 类型的变量，在执行 for 语句之前，显示调用 `Add` 方法，计数器增加 5，表示有五个即将运行的 goroutine。

在 for 语句中，每次循环"创建"一个goroutine，一共创建五个。

如果 WaitGroup 的值大于 0，`Wait` 方法就会阻塞，因此 main goroutine 就算先执行完成，也不能马上退出程序，必须等待 WaitGroup 计数器恢复为 0。

一般会在 goroutine 中，配合使用 defer 调用 `Done`方法，会保证每个 goroutine 一旦执行完成，就调用 `Done` 方法，通知 WaitGroup 该 goroutine 已执行完成。

一旦 WaitGroup 计数变为零，main goroutine 就不会被阻塞，从而继续往下执行，程序退出结束。

执行一下上面的示例代码，看看输出是否符合预期：
```go
Worker 5 starting
Worker 3 starting
Worker 4 starting
Worker 1 starting
Worker 2 starting
Worker 4 done
Worker 1 done
Worker 2 done
Worker 5 done
Worker 3 done
```

符合预期，main goroutine 确实被阻塞了，直到其他 goroutine 都执行完了。

## 引发 panic 的情况
WaitGroup 虽然是开箱即用和并发安全的，但使用时也要注意几点原则，不然可能就引发 panic 了。

不适当地调用 WaitGroup 的 `Done` 方法和 `Add` 方法都可能会导致计数器的值小于零，从而引发 panic：
```
panic: sync: negative WaitGroup counter
```

`Add` 方法比较好理解，因为可以直接传入负数。

另外就是 `Add` 方法的调用，和对 `Wait` 方法的调用如果是同时发起的，比如，放在不同的 goroutine 中并发执行，那么也有可能会引发 panic。

虽然这种情况不太容易复现，因此更加需要重视。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221124095832.png)

## sync.Once

与sync.WaitGroup类型一样，sync.Once类型（以下简称Once类型）也属于结构体类型，同样也是开箱即用和并发安全的。由于这个类型中包含了一个sync.Mutex类型的字段，所以，复制该类型的值也会导致功能的失效。

Once类型的 `Do` 方法只接受一个参数，这个参数的类型必须是`func()`，即：无参数声明和结果声明的函数。

该方法的功能并不是对每一种参数函数都只执行一次，而是只执行“首次被调用时传入的”那个函数，并且之后不会再执行任何参数函数。

所以，如果你有多个只需要执行一次的函数，那么就应该为它们中的每一个都分配一个sync.Once类型的值（以下简称Once值）。

`sync.Once` 通常用于初始化创建实例等场景。
```go
var once sync.Once

var Redis *RedisClient

func ConnectRedis(address string, username string, password string, db int) {
	once.Do(func() {
		Redis = NewClient(address, username, password, db)
	})
}
```

Once类型中有一个名叫 done 的 `uint32` 类型的字段。它的作用是记录其所属值的Do 方法被调用的次数。

Do 方法在功能方面有两个特点：
1. 由于Do 方法只会在参数函数执行结束之后把done字段的值变为1，因此，如果参数函数的执行需要很长时间或者根本就不会结束（比如执行一些守护任务），那么就有可能会导致相关 goroutine 的同时阻塞。
2. Do方法在参数函数执行结束后，对done 字段的赋值用的是原子操作，并且，这一操作是被挂在defer 语句中的。因此，不论参数函数的执行会以怎样的方式结束，done字段的值都会变为1。也就是说，即使这个参数函数没有执行成功（比如引发了一个 panic），我们也无法使用同一个Once值重新执行它了。

## 总结
* sync 代码包的 WaitGroup 类型和 Once 类型都是非常易用的同步工具，它们都是开箱即用和并发安全的
* 使用 WaitGroup 时，应避免出现计数器的值小于0 的情况，否则会引发 panic
* 使用 WaitGroup 时，应遵循 “先统一`Add`，再并发`Done`，最后`Wait`” 这种标准方式，不要在调用`Wait` 方法的同时，并发地通过调用`Add` 方法去增加其计数器的值，因为这也有可能引发 panic

## 参考链接
* [Go 语言核心 36 讲](https://time.geekbang.org/column/intro/100013101)
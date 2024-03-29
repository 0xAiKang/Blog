---
title: Go 语言学习笔记——sync.Mutex与sync
date: 2022-11-08 23:33:41
tags: ["Go"]
categories: ["Go"]
---


这篇笔记主要来介绍 Go 的并发编程相关的核心知识——竞态条件、临界区、同步工具以及死锁。

<!-- more -->

## 竞态条件
**如果两个或者多个 goroutine 在没有互相同步的情况下，访问某个共享的资源，并试图同时读和写这个资源，就处于相互竞争的状态，这种情况被称作竞争条件（race condition），也叫做竞争状态**。

对一个共享资源的读和写操作必须是原子化的，换句话说，同一时刻只能有一个 goroutine 对共享资源进行读和写操作，否则就会出现并发安全问题，会破坏共享数据的一致性。

这种错误一般都很难发现和定位，排查起来的成本也是非常高的，所以一定要尽量避免。

下面通过一段示例代码来加深印象：
```go
var (
	// counter 是一个全局共享变量
	counter int

	// wg 用来等待程序结束
	wg sync.WaitGroup
)

func main() {
	// 计数增加2，表示等待两个 goroutine
	wg.Add(2)

	go incCounter()
	go incCounter()

	// 等待前面的两个 goroutine 结束
	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incCounter() {
	// 保证 incCounter 函数退出时，通知 main goroutine
	defer wg.Done()

	for count := 0; count < 2; count++ {
		// 捕获 counter 的值
		value := counter

		// 当前 goroutine 从线程退出，并放回队列
		runtime.Gosched()
		// 这里使用 runtime.Gosched() 函数，用来模拟发生 I/O 时，线程切换的场景

		// 增加本地 value 变量的值
		value++

		// 将value 赋值给 counter
		counter = value
	}
}
```

示例代码中，使用了 `runtime.Gosched()` 函数，用来模拟发生 I/O 时，线程切换的场景。

关于`sync.WaitGroup` 后面的笔记会详细介绍，这里只需要了解是用来阻塞 main goroutine 执行完直接退出的。

执行上面的示例代码，输出如下：
```go
Final Counter: 2
```

怎么会是 2 呢？\ 
每个 goroutine 各执行两次，一共是四次读写操作，应该是 4 才对呀。

这个就是共享资源引发的并发问题。

那么该如何解决呢？这通常就会涉及同步。

概括来讲，同步的用途有两个，**一个是避免多个线程在同一时刻操作同一个数据块，另一个是协调多个线程，以避免它们在同一时刻执行同一个代码块**。

由于这样的数据块和代码块的背后都隐含着一种或多种资源（比如存储资源、计算资源、I/O 资源、网络资源等等），所以可以把它们看做是共享资源，或者说共享资源的代表。

因此同步其实就是在控制多个线程对共享资源的访问——**一个线程在想要访问某一个共享资源的时候，需要先申请对该资源的访问权限，并且只有在申请成功之后，访问才能真正开始**。

而当线程对共享资源的访问结束时，它还**必须归还对该资源的访问权限**，若要再次访问仍需申请。

## 临界区
可以把这里所说的访问权限想象成一块令牌，线程一旦拿到了令牌，就可以进入指定的区域，从而访问到资源，而一旦线程要离开这个区域了，就需要把令牌还回去，绝不能把令牌带走，因为一旦带走会引发死锁。

如果针对某个共享资源的访问令牌只有一块，那么在同一时刻，就最多只能有一个线程进入到那个区域，并访问到该资源。

这时，可以说，多个并发运行的线程对这个共享资源的访问是完全串行的。

只要一个代码片段需要实现对共享资源的串行化访问，就可以被视为一个**临界区（critical section）**。

比如，在上面的示例代码中，实现了数据块（counter 变量）写入操作的代码就共同组成了一个临界区。如果针对同一个共享资源，这样的代码片段有多个，那么它们就可以被称为**相关临界区**。

## 同步工具
临界区可以是一个内含了共享数据的结构体及其方法，也可以是操作同一块共享数据的多个函数。

临界区总是需要受到保护的，否则就会产生竞态条件。**施加保护的重要手段之一，就是使用实现了某种同步机制的工具，也称为同步工具**。

下面用一张图来更清晰地理解三者之间的关系：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221123171851.png)

在 Go 语言中，同步工具并不少。其中，最重要且最常用的同步工具当属**互斥量（mutual exclusion，简称 mutex）**。sync 包中的Mutex 就是与其对应的类型，该类型的值可以被称为互斥量或者互斥锁。

一个互斥锁可以被用来保护**一个临界区**或者**一组相关临界区**。

因为每当有 goroutine 想进入临界区时，都需要先对它进行锁定，并且，每个 goroutine 离开临界区时，都要及时地对它进行解锁。因此可以通过它来保证，在同一时刻只有一个 goroutine 处于该临界区之内。

下面使用互斥锁来修改前面的示例代码，看看是否可以达到预期结果：

```go
var (
	counter int

	wg sync.WaitGroup

	// 声明一个 Mutex 类型的变量
	mu sync.Mutex
)

func main() {
	wg.Add(2)

	go incCounter()
	go incCounter()

	wg.Wait()
	fmt.Println("Final Counter:", counter)
}

func incCounter() {
  // 互斥量的零值是可用的，因此不需要初始化
	mu.Lock()

	defer func() {
    // 解锁很重要，一定不能忘记
		mu.Unlock()
		wg.Done()
	}()

	for count := 0; count < 2; count++ {
		value := counter

		runtime.Gosched()

		value++

		counter = value
	}
}
```
修改之后的示例代码，声明了一个 `Mutex` 类型的变量，在 `incCounter()` 函数中（临界区），分别加锁 `mu.Lock()` 和解锁 `mu.Unlock` 了。

因为 defer 关键字的存在，解锁操作会在 `incCounter()` 函数调用完成之后，最后去解锁。

运行修改之后的示例代码，输出如下：
```go
Final Counter: 4
```
符合预期。

不过需要注意的是，这里使用互斥锁用来解决示例代码中的原子性问题，并不是最佳的，不能保证绝对的并发安全，至于为什么，以及又该选择什么方式解决原子性问题，可以看这篇笔记。

## 死锁
Go 虽然提供了不少同步工具用来解决竞态条件的问题，但如果使用不当，不但会让程序变慢，还会大大增加死锁（deadlock）的可能性。

所谓的死锁，指的就是当前程序中的main goroutine，以及开发者自己“创建”的 goroutine（这些 goroutine 可以被统称为用户级的 goroutine） 都已经被阻塞。这就相当于整个程序都已经停滞不前了。

Go 语言运行时系统是不允许这种情况出现的，只要它发现所有的用户级 goroutine 都处于等待状态，就会自行抛出一个带有如下信息的 panic：
```
fatal error: all goroutines are asleep - deadlock!
```

**注意，这种由 Go 语言运行时系统自行抛出的 panic 都属于致命错误，都是无法被恢复的，调用recover 函数对它们起不到任何作用。也就是说，一旦产生死锁，程序必然崩溃。**

因此，一定要尽可能避免死锁的发生。而最简单、有效的方式就是让每一个互斥锁都只保护一个临界区或一组相关临界区。

而且，对于同一个 goroutine 而言，既不要重复锁定一个互斥锁，也不要忘记对它进行解锁。

因此通常会在 `mu.Lock()` 操作的后面紧跟一个 `defer Unlock()`，这是最保险的一种做法。

## 总结
* 互斥锁常常被用来：保证多个 goroutine 并发地访问同一个共享资源时的完全串行
* 不要重复锁定互斥锁
* 不要忘记解锁互斥锁，通常配合使用 defer 语句解锁
* 不要对尚未锁定的互斥锁解锁
* 不要在多个函数之间直接传递互斥锁
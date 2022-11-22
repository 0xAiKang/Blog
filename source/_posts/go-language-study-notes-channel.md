---
title: Go 语言学习笔记——channel
date: 2022-11-07 10:33:41
tags: ["Go"]
categories: ["Go"]
---

如果 main goroutine 退出了，那么也意味着整个应用程序的退出。

此外，还要注意的是，goroutine 执行的函数或方法即便有返回值，Go 也会忽略这些返回值。所以，如果要获取 goroutine 执行后的返回值，需要另行考虑其他方法，比如通过 goroutine 间的通信来实现。

<!-- more -->

## goroutine 间的通信
传统的编程语言（C++、Java、Python 等）并非面向并发而生的，所以它们面对并发的逻辑多是基于操作系统的线程。

线程之间的通信，利用的也是操作系统提供的线程或进程间通信的原语，比如：共享内存、信号（signal）、管道（pipe）、消息队列、套接字（socket）等。

在这些通信原语中，使用最多、最广泛的（也是最高效的）是结合了线程同步原语（比如：锁以及更为低级的原子操作）的共享内存方式，因此，我们可以说传统语言的并发模型是基于对内存的共享的。

Go 语言从设计伊始，就将解决上面这个传统并发模型的问题作为 Go 的一个目标，并在新并发模型设计中借鉴了著名计算机科学家Tony Hoare提出的 CSP（Communicationing Sequential Processes，通信顺序进程）并发模型。

简单看下 CSP 的通信模型示意图：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/CSP模型.png)

在 Go 中，与“Process”对应的是 goroutine。为了实现 CSP 并发模型中的输入和输出原语，Go 还引入了 goroutine（P）之间的通信原语channel。

goroutine 通过 channel 获取输入数据，再将处理后得到的结果通过 channel 输出。通过 channel 将 goroutine（P）组合连接在一起，让设计和编写大型并发系统变得更加简单和清晰。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/goroutine并发模型.png)

比如上面提到的获取 goroutine 的退出状态，就可以使用 channel 原语实现：
```go
func spawn(f func() error) <-chan error {
    c := make(chan error)
    go func() {
        c <- f()
    }()
    return c
}

func main() {
    c := spawn(func() error {
        time.Sleep(2 * time.Second)
        return errors.New("timeout")
    })
    fmt.Println(<-c)
}
```

运行上面的实例代码，输出如下：
```go
// 延迟 2s 打印
timeout
```

该示例在 main.goroutine 与子 goroutine 之间建立一个元素类型为 error 的 channel，子 goroutine 退出时，会将它执行的函数的错误值写入这个 channel，main.goroutine 可以通过读取 channel 的值来获取子 gotouine 的退出状态。

## 创建 channel
channel 也是一等公民。

可以像使用普通变量那样使用 channel，定义 channel 类型变量，给 channel 变量赋值，将 channel 作为参数传递给函数 / 方法、将 channel 作为返回值从函数 / 方法中返回，甚至将 channel 发送到其他 channel 中。

和切片、结构体、map 等一样，channel 也是一种复合数据类型。复合数据类型，在声明类型变量时，必须给出具体的元素类型。

```go
var ch chan int
```

上面的示例代码中，声明了一个元素为 int 类型的 channel 类型变量 ch。

如果 channel 类型变量在声明时没有被赋予初值，那么它的默认值为 nil。

和其他复合类型不同的是，给 channel 类型变量赋初值的唯一方式就是 make 这个 Go 预定义函数：
```go
ch1 := make(chan int)
ch2 := make(chan int, 5)
```

ch1 表示元素类型为 int 的 channel 类型，是**无缓冲 channel**; ch2 表示元素类型为 int 的 channel 类型，**带缓冲 channel**，且缓冲区长度为 5。

这两种类型变量关于发送（send）和接收（receive）的特性是不同的，下面基于这两种类型的 channel，看看 channel 类型变量如何进行发送和接收数据元素。

## 发送与接收

Go 提供了<-操作符用于对 channel 类型变量进行发送与接收操作：
```go
ch1 <- 13     // 将整型字面值 13 发送到无缓冲 channel 类型变量 ch1 中
n := <- ch1   // 将无缓冲 channel 类型变量 ch1 中接收的整型值存储到整型变量 n 中
ch2 <- 17     // 将整型字面值 17 发送到带缓冲 channel 类型变量 ch2 中
m := <- ch2   // 从带缓冲 channel 类型变量 ch2 中接收一个整型值存储到整型变量 m 中
```

在理解 channel 的发送与接收操作时，你一定要始终牢记：**channel 是用于 Goroutine 间通信的**，所以绝大多数对 channel 的读写都被分别放在了不同的 Goroutine 中。

### 无缓冲 channel

由于无缓冲 channel 的运行时层实现不带有缓冲区，所有 goroutine 对无缓冲 channel 的接收和发送是同步的。

也就是说，对同一个无缓冲 channel，只有对它进行接收操作的 Goroutine 和对它进行发送操作的 Goroutine 都存在的情况下，通信才能得以进行，可以结合 goroutine 并发模型理解：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/goroutine并发模型.png)

否则单方面的操作会让对应的 Goroutine 陷入挂起状态，比如下面示例代码：

```go
func main() {
    ch1 := make(chan int)
    ch1 <- 13 // fatal error: all goroutines are asleep - deadlock!
    n := <-ch1
    println(n)
}
```

在上面的示例中，创建了一个无缓冲 channel 类型变量 ch1，对 ch1 的读和写都放在一个 goroutine 中了（因为这里没有手动创建 goroutine，因此只有 main goroutine），因此陷入挂起状态了，这也是上面提到的，为什么要把对 channel 的读写放在不同的 goroutine 中的原因。

解决办法也很简单，只需要将接口操作或者发送操作放到另外一个 goroutine 中就可以了。

```go
func main() {
    ch1 := make(chan int)
    go func() {
        ch1 <- 13 // 将发送操作放入一个新goroutine中执行
    }()
    n := <-ch1
    println(n)
}
```

由此，可以得出结论：**对无缓冲 channel 类型的发送与接收操作，一定要放在两个不同的 Goroutine 中进行，否则会导致 deadlock（死锁）**。

### 缓冲带 channel
和无缓冲 channel 相反，带缓冲 channel 的运行时层面实现带有缓冲区，因此，**对带缓冲 channel 的发送操作在缓冲区未满、接收操作在缓冲区非空的情况下是异步的**（发送或接收无需阻塞等待）。

也就是，下面四种情况（仅针对 带缓冲 channel）：
1. 如果缓冲区已满，进行发送操作，会导致 goroutine 挂起
2. 如果缓冲区未满，进行发送操作，不会导致 goroutine 阻塞挂起
3. 如果缓冲区为空，进行接收操作，会导致 goroutine 阻塞挂起
4. 如果缓冲区有数据，进行接收操作，不会导致 goroutine 阻塞挂起

可以结合下面的示例代码理解：
```go
package main

func main() {
	ch2 := make(chan int, 1)
	// 从ch2 的缓冲区接收数据
	n := <-ch2   // 因为此时ch2 的缓冲区中无数据，因此将会导致 goroutine 挂起
	fmt.Println(n)

	ch3 := make(chan int, 1)
	ch3 <- 17 // 向 ch3 发送一个整型数
	ch3 <- 18 // 由于此时ch3中缓冲区已满，再向ch3发送数据也将导致 goroutine 挂起
}
```

也正是因为带缓冲 channel 与无缓冲 channel 在发送与接收行为上的差异，在具体使用上，它们有各自的“用武之地”，这个我们等会再细说，现在我们先继续把 channel 的基本语法讲完。

使用操作符`<-`，还可以声明只发送 channel 类型（send-only）和只接收 channel 类型（recv-only），接着看下面这个例子：
```go
ch1 := make(chan<- int, 1) // 只发送channel类型
ch2 := make(<-chan int, 1) // 只接收channel类型
<-ch1       // invalid operation: <-ch1 (receive from send-only type chan<- int)
ch2 <- 13   // invalid operation: ch2 <- 13 (send to receive-only type <-chan int)
```

可以从上面的示例代码中看到，试图从一个只发送 channel 类型变量中接收数据，或者向一个只接收 channel 类型发送数据，都会导致编译错误。

通常只发送 channel 类型和只接收 channel 类型，会被用作函数的参数类型或返回值，用于限制对 channel 内的操作，或者是明确可对 channel 进行的操作的类型，比如下面这个例子：

```go
func produce(ch chan<- int) {
    for i := 0; i < 10; i++ {
        ch <- i + 1
        time.Sleep(time.Second)
    }
    close(ch)
}
func consume(ch <-chan int) {
    for n := range ch {
        println(n)
    }
}
func main() {
    ch := make(chan int, 5)
    var wg sync.WaitGroup
    wg.Add(2)
    go func() {
        produce(ch)
        wg.Done()
    }()
    go func() {
        consume(ch)
        wg.Done()
    }()
    wg.Wait()
}
```

在这个例子中，分别启动了两个 goroutine，分别代表生产者（produce）和消费者（consume）。

生产者只能向 channel 中发送数据，使用 `chan<- int` 作为 produce 函数的参数类型。\
消费者只能从 channel 中接收数据，使用 `int<- chan` 作为 consume 函数的参数类型。

在消费者函数中，使用 `for range` 从 channel 中接收数据，`for range` 会阻塞在对 channel 的接收操作，直到 **channel 中有数据可以接收**或者**channel 被关闭循环**，才会继续向下执行。channel 被关闭后，for range 循环也就结束了。

## 关闭 channel
在上面的例子中，produce 函数在发送完数据后，调用 Go 内置的 close 函数关闭了 channel。channel 关闭后，所有等待从这个 channel 接收数据的操作都将返回。

采用不同接收语法形式的语句，在 channel 被关闭后的返回值的情况：
* `n := <- ch`：当ch被关闭后，n将被赋值为ch元素类型的零值，无法准确判断 channel 是否被关闭
* `m, ok := <-ch`：当ch被关闭后，m将被赋值为ch元素类型的零值, ok值为false，可以准确判断 channel 是否被关闭
* `for v := range ch`：当ch被关闭后，for range循环结束，可以准确判断 channel 是否被关闭


另外，从上面的示例中还可以看到，channel 是在 produce 函数中被关闭的，这也是 channel 的一个使用惯例，那就是**发送端负责关闭 channel**。

这里为什么要在发送端关闭 channel 呢？

这是因为发送端没有像接受端那样的、可以安全判断 channel 是否被关闭了的方法（上面的两种方式）。同时，一旦向一个已经关闭的 channel 执行发送操作，这个操作就会引发 panic：
```go
ch := make(chan int, 5)
close(ch)
ch <- 13 // panic: send on closed channel
```

## select
当涉及同时对多个 channel 进行操作时，可以使用 Go 为 CSP 并发模型提供的另外一个原语 select。

通过 select，可以同时在多个 channel 上进行发送 / 接收操作：
```go
func main() {
	ch1 := make(chan int, 1)
	ch2 := make(chan int, 1)
	ch3 := make(chan int, 1)

	ch1 <- 11
	ch2 <- 12
	ch3 <- 13
	select {
	// 从 channel ch1 接收数据
	case x := <-ch1:
		println(x)
	// 从channel ch2接收数据，并根据ok值判断ch2是否已经关闭
	case y, ok := <-ch2:
		println(ok)
		println(y)
	// 从 channel ch3 接收数据
	case z := <-ch3:
		println(z)
	// 当上面case中的channel 均无法接收数据时，执行该默认分支
	default:
		println("default")
	}
}
```
这里先简单了解一下基本语法，后面再详细讲解。

## 无缓冲带 channel 惯用法

无缓冲 channel 兼具通信和同步特性，在并发程序中应用颇为广泛。现在我们来看看几个无缓冲 channel 的典型应用。

### 用作信号传递

无缓冲 channel 用作信号传递的时候，有两种情况，分别是 1 对 1 通知信号和 1 对 n 通知信号。

#### 1 对 1 通知信号

```go
type signal struct{}
func worker() {
    println("worker is working...")
    time.Sleep(1 * time.Second)
}
func spawn(f func()) <-chan signal {
    c := make(chan signal)
    go func() {
        println("worker start to work...")
        f()
        c <- signal(struct{}{})
    }()
    return c
}
func main() {
    println("start a worker...")
    c := spawn(worker)
    <-c
    fmt.Println("worker work done!")
}
```

运行上面的示例代码，输出以下结果：
```go
start a worker...
worker start to work...
worker is working...
worker work done!
```

这里之所以会执行 worker 函数（worker 函数是在 新的goroutine内的）。

spawn 函数返回的 channel 相当于是一个新 goroutine 创建的“通知信号”，利用无缓冲channel 的特性，对无缓冲 channel 的接收和发送操作是同步的，只有同时具备接收和发送能力才会继续往下执行，因此一定是新的 goroutine 先执行完成，然后才是 main goroutine 执行完成。

#### 1 对 n 通知信号

有些时候，无缓冲 channel 还被用来实现 1 对 n 的信号通知机制。这样的信号通知机制，常被用于协调多个 Goroutine 一起工作，比如下面的例子：
```go
type signal struct{}

func worker(i int) {
	fmt.Printf("worker %d: is working...\n", i)
	time.Sleep(1 * time.Second)
	fmt.Printf("worker %d: works done\n", i)
}

func spawnGroup(f func(i int), num int, groupSignal <-chan signal) <-chan signal {
	c := make(chan signal)
	var wg sync.WaitGroup
	for i := 0; i < num; i++ {
		wg.Add(1)
		go func(i int) {
			<-groupSignal
			fmt.Printf("worker %d: start to work...\n", i)
			f(i)
			wg.Done()
		}(i + 1)
	}
	go func() {
		wg.Wait()
		c <- signal(struct{}{})
	}()
	return c
}

func main() {
	fmt.Println("start a group of workers...")
	groupSignal := make(chan signal)
	time.Sleep(5 * time.Second)
	c := spawnGroup(worker, 5, groupSignal)
	fmt.Println("the group of workers start to work...")
	close(groupSignal)
	<-c
	fmt.Println("the group of workers work done!")
}
```
在这个例子中，main goroutine 创建了一组 5 个 worker goroutine，这些 Goroutine 启动后会阻塞在名为 groupSignal 的无缓冲 channel 上。

main goroutine 通过 close(groupSignal)向所有 worker goroutine 广播“开始工作”的信号，收到信号后，所有 worker goroutine 会“同时”开始工作，也就打印出了结果。

运行上面的示例代码，输出以下结果：
```go
start a group of workers...
the group of workers start to work...
worker 1: start to work...
worker 1: is working...
worker 2: start to work...
worker 2: is working...
worker 3: start to work...
worker 3: is working...
worker 4: start to work...
worker 4: is working...
worker 5: start to work...
worker 5: is working...
worker 5: works done
worker 2: works done
worker 1: works done
worker 3: works done
worker 4: works done
the group of workers work done!
```

### 替代锁机制
无缓冲 channel 具有同步特性，这让它在某些场合可以替代锁，让我们的程序更加清晰，可读性也更好。我们可以对比下两个方案，直观地感受一下。

首先看下传统基于“共享内存”+“互斥锁”的 Goroutine 安全的计数器的实现：
```go
type counter struct {
    sync.Mutex
    i int
}
var cter counter
func Increase() int {
    cter.Lock()
    defer cter.Unlock()
    cter.i++
    return cter.i
}
func main() {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            v := Increase()
            fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

在这个示例中，使用了一个带有互斥锁保护的全局变量作为计数器，所有要操作计数器的 Goroutine 共享这个全局变量，并在互斥锁的同步下对计数器进行自增操作。

接下来再看更符合 Go 设计惯例的实现，也就是使用无缓冲 channel 替代锁后的实现：
```go
type counter struct {
    c chan int
    i int
}
func NewCounter() *counter {
    cter := &counter{
        c: make(chan int),
    }
    go func() {
        for {
            cter.i++
            cter.c <- cter.i
        }
    }()
    return cter
}
func (cter *counter) Increase() int {
    return <-cter.c
}
func main() {
    cter := NewCounter()
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            v := cter.Increase()
            fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

在这个示例中，将计数器操作全部交给一个独立的 Goroutine 去处理，并通过无缓冲 channel 的同步阻塞的特性，实现计数器的控制。\
这样其他 Goroutine 通过 Increase 函数试图增加计数器值的动作，实质上就转化为了一次无缓冲 channel 的接收动作。

这种并发设计逻辑更符合 Go 语言所倡导的**“不要通过共享内存来通信，而是通过通信来共享内存”**的原则。

运行上面的示例代码，可以得到和互斥锁方案相同的输出：
```go
goroutine-9: current counter value is 10
goroutine-0: current counter value is 1
goroutine-6: current counter value is 7
goroutine-2: current counter value is 3
goroutine-8: current counter value is 9
goroutine-4: current counter value is 5
goroutine-5: current counter value is 6
goroutine-1: current counter value is 2
goroutine-7: current counter value is 8
goroutine-3: current counter value is 4
```

## 带缓冲 channel 的惯用法
带缓冲的 channel 与无缓冲的 channel 最大的不同之处， 就在于它的异步性。

对一个带缓冲的 channel，在缓冲区未满的情况下，对它进行发送操作的 Goroutine 不会阻塞挂起; 在缓冲区有数据的情况下，对他进行接收操作的 Goroutine 也不会阻塞挂起。

* 无论是 1 收 1 发还是多收多发，带缓冲 channel 的收发性能都要好于无缓冲 channel；
* 对于带缓冲 channel 而言，发送与接收的 Goroutine 数量越多，收发性能会有所下降；
* 对于带缓冲 channel 而言，选择适当容量会在一定程度上提升收发性能。

### 用作计数信号量

Go 并发设计的一个惯用法，就是将带缓冲 channel 用作计数信号量（counting semaphore）。

带缓冲 channel 中的当前数据个数代表的是，当前同时处于活动状态（处理业务）的 Goroutine 的数量，而带缓冲 channel 的容量（capacity），就代表了允许同时处于活动状态的 Goroutine 的最大数量。

向带缓冲 channel 的一个发送操作表示获取一个信号量，而从 channel 的一个接收操作则表示释放一个信号量。

```go
var active = make(chan struct{}, 3)
var jobs = make(chan int, 10)
func main() {
    go func() {
        for i := 0; i < 8; i++ {
            jobs <- (i + 1)
        }
        close(jobs)
    }()
    var wg sync.WaitGroup
    for j := range jobs {
        wg.Add(1)
        go func(j int) {
            active <- struct{}{}
            log.Printf("handle job: %d\n", j)
            time.Sleep(2 * time.Second)
            <-active
            wg.Done()
        }(j)
    }
    wg.Wait()
}
```

运行示例代码，输出如下：
```go
2022/11/11 10:03:07 handle job: 2
2022/11/11 10:03:07 handle job: 8
2022/11/11 10:03:07 handle job: 1
2022/11/11 10:03:09 handle job: 3
2022/11/11 10:03:09 handle job: 4
2022/11/11 10:03:09 handle job: 6
2022/11/11 10:03:11 handle job: 7
2022/11/11 10:03:11 handle job: 5
```

从示例运行结果中的时间戳中，可以看到，虽然创建了很多 Goroutine，但由于计数信号量的存在，同一时间内处理活动状态（正在处理 job）的 Goroutine 的数量最多为 3 个。

## 总结
* channel 是用于 goroutine 间通信的
* 通过预定义函数 make，可以创建两类 channel
* 无缓冲 channel 具备**通信与同步特性**，常用于作为信号通知或替代同步锁
* 带缓冲 channel 具备**异步性**，更适合用来实现基于内存的消息队列、计数信号量等

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
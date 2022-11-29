---
title: Go 语言学习笔记——如何理解 channel 的无缓冲带和有缓冲带
date: 2022-11-11 21:44:51
tags: ["Go"]
categories: ["Go"]
---

channel 是否有缓冲带，其行为会有一些不同。理解这个差异对决定到底应该使不使用缓冲带很有帮助。

<!-- more -->

## 理解无缓冲带 channel

使用无缓冲带 channel 进行接收和发送操作时，一定要注意，接收和发送操作不能放在同一个 goroutine 中，否则是没有意义的。

为什么这么说呢？\
这是因为无缓冲带 channel 的特点是，**同步阻塞**。

无缓冲带，可以分为两种情况：
1. 从无缓冲带 channel 接收数据时，如果同时没有发送者发送数据，那么则会一直阻塞，直到有发送者出现进行发送。
2. 向无缓冲带 channel 发送数据，如果同时没有接收者进行接收，那么则会一直阻塞，直到有接收者出现进行接收。

结合下面这个图进行理解，也就是无缓冲带 channel 只有同时具备接收和发送能力才会继续往下执行。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221129174724.png)

结合下面的示例代码来理解：
```go
func main() {
	// 无缓冲带
	ch1 := make(chan int)
	ch2 := make(chan int)

	ch1 = ch2
	go func() {
		fmt.Println("go")
		ch2 <- 123

		i := <-ch1
		fmt.Println(i)
	}()

	time.Sleep(10 * time.Second)
	fmt.Println("done")
}
```

也就是无论 time.Sleep 休息多少秒， goroutine 中的 123 都不会打印出来。因为 goroutine  阻塞在 `ch2 <- 123` 这行代码了。

那么应该怎么改写， 123 才会打印出来呢？

只需要将发送数据这一行代码移出 goroutine 即可：
```go
...
go func() {
  fmt.Println("go")

  i := <-ch1
  fmt.Println(i)
}()

ch2 <- 123
```

运行上面的示例代码，输出如下：
```go
go
123
done
```

符合预期。

## 理解有缓冲带 channel
很多资料或者文章里面会说，有缓冲带 channel 是**异步非阻塞**的，这个异步到底该如何理解呢？

有缓冲带，可以分为四种情况：
1. 如果缓冲带已满，那么对它的所有**发送操作都会被阻塞**，直到缓冲带中有元素值被接收
2. 如果缓冲带已空，那么对它的所有**接收操作都会被阻塞**，直到通道中有新的元素值出现
3. 如果缓冲带未满，那么对它的所有**发送操作都不会阻塞**
4. 如果缓冲带非空，那么对它的所有**接收操作都不会阻塞**

上面所说异步非阻塞，就是针对第三四种情况的，而第一二种情况，在缓冲带已满、已空时，就和无缓冲带一样了，因此也会是同步阻塞的。

下面用一段示例代码来加深理解。
还是上面的示例代码，只是初始化 channel 时，使用的是有缓冲带的 channel。
```go
func main() {
	ch1 := make(chan int, 1)
	ch2 := make(chan int, 1)

	ch1 = ch2
	go func() {
		fmt.Println("go")
		ch2 <- 123

		i := <-ch1
		fmt.Println(i)
	}()

	time.Sleep(1 * time.Second)
	fmt.Println("done")
}
```

执行一下，输出如下：
```
go
123
done
```

为什么现在就打印出 123 来了？\
正是因为有缓冲带的存在，有缓冲带 channel 不需要同时具有接收和发送能力，才能继续往下执行，数据的发送者将数据放在缓冲带即可，而无需等待数据的接收者出现。

结合下面这个图进行理解：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221129174810.png)

在这个过程里面，两个操作不是同步的，因此就说有缓冲带 channel 是异步非阻塞。

这两种类型的缓冲带，各自有各自的特点，有的场景仅适合使用无缓冲带 channel，有的场景则只适合使用有缓冲带 channel，不过，也有两种缓冲带都可以使用的场景。

## 一个示例
下面这个示例就是两种缓冲带都可以使用的场景。

先来看看示例代码：
```go
func hello(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintln(w, "hello go")
}

func main() {
	http.HandleFunc("/hello", hello)

	keepPrinting()

	fmt.Println("App runs on port 9999.")
	log.Fatal(http.ListenAndServe(":9999", nil))
}

func keepPrinting() {
	i := 0
	for {
		i++
		log.Println("i = ", i)
  }
}

```

为了方便后面 goroutine 的使用，这里启动了一个 HTTP 服务。

函数 keepPrinting 的作用很简单，在一个不会主动停止的 for 循环中，一直打印变量 i 的值。

运行这段示例代码，会发现终端一直有输出：
```go
...
2022/11/25 09:59:37 i =  243979
2022/11/25 09:59:37 i =  243980
2022/11/25 09:59:37 i =  243981
2022/11/25 09:59:37 i =  243982
2022/11/25 09:59:37 i =  243983
...
```

这时，访问一下刚刚启动的 Web 服务：
```bash
$ curl -I http://localhost:9999/hello
HTTP/1.1 502 Bad Gateway
```

怎么状态码是 502，不是 200？\
这是因为 main goroutine 一直在 for 循环里面，就没有出来过，所以服务压根就没有启动。

那么有没有什么办法，既可以保证服务正常启动，同时又可以执行 keepPrinting 函数？

答案是有的，那就是“创建”一个 goroutine，由这个 goroutine 去执行 keepPrinting 函数。

因此，上面的示例代码就变成了这样：
```go
func main() {
	http.HandleFunc("/hello", hello)

    go keepPrinting()

	fmt.Println("App runs on port 9999.")
	log.Fatal(http.ListenAndServe(":9999", nil))
}
```

运行更新之后的示例代码，会发现终端同样一直有输出：
```go
...
2022/11/25 09:59:37 i =  243979
2022/11/25 09:59:37 i =  243980
2022/11/25 09:59:37 i =  243981
2022/11/25 09:59:37 i =  243982
2022/11/25 09:59:37 i =  243983
...
```

访问一下 Web 服务：
```bash
$ curl -I http://localhost:9999/hello
HTTP/1.1 200 OK
```

状态码是 200，说明服务已经启动了。

现在虽然达到了上面的预期结果，但是终端输出的内容太多了，导致其他重要的内容容易一下子被“冲走了”。

那么有没有什么办法，可以限制一下输出，只在服务端收到来自客户端的请求时，才会去打印。

当然是可以的，这里就要借助 channel 了。

对上面的示例代码进行改造：
```go

var ch chan string

func hello(w http.ResponseWriter, req *http.Request) {
	ch <- req.URL.Path
	fmt.Fprintln(w, "hello go")
}

func main() {
  // 两种都可以使用
	// ch = make(chan string)
  ch = make(chan string, 1)
	http.HandleFunc("/hello", hello)

  go keepPrinting()

	fmt.Println("App runs on port 9999.")
	log.Fatal(http.ListenAndServe(":9999", nil))
}

func keepPrinting() {
	i := 0
	for {
		i++
		log.Println("i = ", i)
		url := <-ch
		log.Println("URL.Path = ", url)
	}
}
```

再次执行示例代码，启动 Web 服务：
```bash
App runs on port 9999.
2022/11/25 10:55:22 i =  1
```

终端访问 Web 服务：
```bash
$ curl -I http://localhost:9999/hello
HTTP/1.1 200 OK
```

控制台继续输出如下内容：
```bash
2022/11/25 10:55:56 URL.Path =  /hello
2022/11/25 10:55:56 i =  2
```

这里这个示例，无论是使用无缓冲 channel还是有缓冲 channel，其效果都是一样的。
1. 使用无缓冲 channel 时，对无缓冲 channel 的接收和发送操作是同步的，只有同时具备接收和发送能力才会继续往下执行。
2. 使用有缓冲 channel 时，因为在缓冲带已空的情况下，对它的所有接收操作都会被阻塞，因此效果上和使用无缓冲 channel 是一样的。

当没有新的请求进来时，也就是没有对 channel 进行发送操作，goroutine 会一直阻塞在 `url := <-ch` 对 channel 的接收操作上。

一旦新的请求进来了，也就是对 channel 进行发送操作了，此时 goroutine 通过接收操作拿到了数据，因此继续往下执行。

goroutine 再次重新进入 for 循环，阻塞等待，重复上面的逻辑。

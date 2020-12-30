---
title: 理解 Go 语言中的字符串和字节数组
date: 2020-12-30 23:57:13
tags: ["Go"]
categories: ["Go"]
---

最近在学习Go 语言时，遇到一个很有意思的问题，记录一下。

第一次使用`redisgo` 时，很懵，怎么取出来的数据跟我存的完全不一样？
```
package main

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
)

func main() {
  conn, _ := redis.Dial("tcp", "127.0.0.1:6379")
  defer conn.Close()
  
  conn.Send("SET", "hello", "hello")
  conn.Send("GET", "hello")
  conn.Flush()
  
  v, _ := conn.Receive()
	fmt.Println(v)
}
```

打印结果：
```
[104 101 108 108 111]
```

当看到这个打印结果时，我是很懵的，我明明存进去的是一个 `hello`，怎么取出来却成了一个数组？

要回答这个问题，就得了解Go 语言中的字符串这个数据结构了。

### 认识字符串
字符串是Go 语言中最常用的基础数据类型之一，虽然字符串往往是被看作是一个整体，但实际上字符串是一块连续的内存空间，也可以理解成是一个由字符组成的数组。

字符串虽然在 Go 语言中是基本类型 string（`hello`），但是它其实就是字符组成的数组（`[104 101 108 108 111]`）。

作为数组来说，它会占用一片连续的内存空间，这片连续的内存空间就存储了一些**字节**，这些字节共同组成了字符串。

**Go 语言中的字符串是一个只读的字节数组切片**。

尝试将数组切片转换成字符串：
```
package main

import "fmt"

func main()  {
  // 注意这里的数据类型是 uint8，而不是 int、uint
	sli := []uint8{104, 101, 108, 108, 111}
	fmt.Println(sli)
	fmt.Printf("%T \n", sli)
	
	fmt.Println(string(sli))       
	fmt.Printf("%T \n", string(sli))
}
```

打印结果：
```
[104 101 108 108 111]
[]uint8 
hello
string 
```

来看看`hello`这个字符串在内存中的存储方式：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20201230160415.png)

你可能会问：`0x68`、`0x65`、`0x6c`、`0x6c`、`0x6f`这些东西是什么？

他们是`hello` 这个字符串的切片数组的每一个数字所对应的十六进制：
* 104 => `0x68`
* 101 => `0x65`
* 108 => `0x6c`
* 108 => `0x6c`
* 111 => `0x6f`

### 声明字符串

在Go 语言中，有两种字面量方式可以声明一个字符串，一种是使用双引号，另一种则是使用反引号：
```
str1 := "this is a string"
str2 := `this is another`
```

使用双引号声明的字符串其实和其他语言中的字符串声明没有太多区别，它只能用于简单、单行的字符串。

并且如果字符串内部出现双引号时需要使用 `\` 符号来避免编译器解析错误，而使用反引号则可以很好的摆脱这一限制。

在遇到需要写 JSON 或者其他数据格式的场景下非常方便，下面两个 JSON 字符串的写法都没问题，但显然第二种方式更简洁、自然、便于阅读。
```
str1 := "{\"page\": 1, \"fruits\": [\"apple\", \"pear\"]}"
str2 := `{"page": 1, "fruits": ["apple", "pear"]}`
```

### 参考链接
* [谈 Golang 中的字符串和字节数组](https://www.infoq.cn/article/wj08lvwzu6tnkv4sidy6)
---
title: Go 语言学习笔记——map
date: 2022-10-27 23:25:51
tags: ["Go"]
categories: ["Go"]
---

作为 Go 语言复合类型之一的字典 map，使用频率也是较高的。

<!-- more -->

很多中文 Go 编程语言类技术书籍都会将它翻译为映射、哈希表或字典，在这篇笔记中约定使用 map。

## map
map 是 Go 语言提供的一种抽象数据类型，用于实现特定键值的快速查找与更新，它表示一组无序的键值对，map 中的每个 key 都是唯一的，并且有与之对应的一个 value。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221107232209.png)

和切片类似，作为复合类型的 map，它在 Go 中的类型表示也是由 key 类型与 value 类型组成的，就像下面代码：

```go
map[key_type]value_type
```

和数组一样，如果两个 map 类型的 key 元素类型相同，value 元素类型也相同，那么我们可以说它们是同一个 map 类型，否则就是不同的 map 类型。

```go
func foo(map[int]int) {}

func main() {
	var m1 map[int]int
	var m2 map[int]string

	foo(m1)    // 正常编译
	foo(m2)    // 编译失败：map[int]string 与函数foo参数的类型map[int]int 不是同一map类型
}
```

需要注意的是 map 虽然对 value 没有限制，但是对 key 的类型有严格的要求。
因为需要保证 key 的唯一性，key 类型就必须支持 `==` 和 `!=` 这两种比较运算符。

> 那么有哪些类型不能作为 map 的 key 类型呢？

答案是**函数类型、map 类型以及切片类型**。

可以来做一个实验：
```go
s1 := make([]int, 1)
s2 := make([]int, 2)
f1 := func() {}
f2 := func() {}
m1 := make(map[int]string)
m2 := make(map[int]string)
println(s1 == s2) // 编译失败：invalid operation: s1 == s2 (slice can only be compared to nil)
println(f1 == f2) // 编译失败：invalid operation: f1 == f2 (func can only be compared to nil)
println(m1 == m2) // 编译失败：invalid operation: m1 == m2 (map can only be compared to nil)
```

可以看到，这三种类型直接进行比较时，会编译失败。

## map 的声明与初始化

可以这样声明一个 map 变量：
```go
var m map[string]int   // 声明一个 map[string]int 类型的变量
```

和切片类型变量一样，如果没有显示赋予 map 变量初始值， map 类型变量的默认值就是 `nil`。

不过不同的是，初始值为 `nil` 的切片，可以借助 append 函数对其进行操作。
而 map 因为自身其复杂的实现方式，无法“零值可用”。\
所以，如果直接对处于零值的 map 进行操作，就会导致运行时异常（panic），从而导致程序进程异常退出：

```go
var m map[string]int // m = nil
m["key"] = 1         // 发生运行时异常：panic: assignment to entry in nil map
```

所以，在使用 map 之前，必须先对其进行初始化，初始化有两种方式，下面一一说明。

### 复合字面值初始化

```go
m := map[int]int{}
```
和前面声明 map 很像，不过有两点不同：`var` 关键字替换成了 `:=`，其次就是 value 类型后面多了一对花括号。

虽然此时 map 类型变量 m 中没有任何键值对，但变量 m 也不等同于初值为 nil 的 map 变量。

再次进行操作，就不会引发运行异常。
```go
m := map[int]int{}
m[99] = 99
println("m[99] = ", m[99])  // m[99] = 99
```

对于稍微复杂一些的复合字面值，可以使用 Go 语言提供的“语法糖”省略部分字面值：
```go
type Position struct { 
    x float64 
    y float64
}

// 正常编译，但写法比较臃肿
m1 := map[Position]string{
    Position{29.935523, 52.568915}: "school",
    Position{25.352594, 113.304361}: "shopping-mall",
    Position{73.224455, 111.804306}: "hospital",
}

// 同样正常编译，通过使用语法糖直接省略 key 类型
m2 := map[Position]string{
    {29.935523, 52.568915}: "school",
    {25.352594, 113.304361}: "shopping-mall",
    {73.224455, 111.804306}: "hospital",
}

// 不过需要注意，对 map 进行插入操作时，不能省略否则会语法错误
m2[Position{22.935523, 34.568915}] = "store"   // 正常编译
m2[{22.935523, 34.568915}] = "store"           // 语法错误
```

### make 函数

和切片一样，通过 make 的初始化方式，我们可以为 map 类型变量指定键值对的初始容量，但无法进行具体的键值对赋值：
```go
m1 := make(map[int]string)     // 未指定初始容量
m2 := make(map[int]string, 8)  // 指定初始容量为8
```

不过，map 类型的容量不会受限于它的初始容量值，当其中的键值对数量超过初始容量后，Go 运行时会自动增加 map 类型的容量，保证后续键值对的正常插入。

## map 基本操作
因为 map 是 Go 语言中十分常用的复合数据类型，所以下面来一一了解下常用的操作有哪些。

### 插入操作

面对一个非 nil 的 map 类型变量，可以插入**符合 map 类型定义**的任意键值对。

插入新键值对的方式很简单，我们只需要把 value 赋值给 map 中对应的 key 就可以了：
```go
m := make(map[int]string)
m[1] = "value1"
m[2] = "value2"
m[3] = "value3"

// 如果对一个已经存在的 key 进行插入操作会覆盖原值
m[1] = "new value1"
// 此时 m 为 map[1:new value1 2:value2 3:value3]
```

Go 运行时会负责 map 变量内部的内存管理，因此除非是系统内存耗尽，否则不用担心向 map 中插入新数据的数量和执行结果。

### 获取键值对数量

切片可以通过 len 函数获取其长度，map 也可以通过内置函数 len，获取当前变量已经存储的键值对数量：
```go
m := map[string]int {
  "key1" : 1,
  "key2" : 2,
}
fmt.Println(len(m)) // 2
m["key3"] = 3  
fmt.Println(len(m)) // 3
```

不过，这里要注意的是我们不能对 map 类型变量调用 cap，来获取当前容量，这是 map 类型与切片类型的一个不同点。

### 查找和数据读取

和写入相比，map 类型更多用在查找和数据读取场合。\
所谓查找，就是判断某个 key 是否存在于某个 map 中。有了前面向 map 插入键值对的基础，可能自然而然地想到，可以用下面代码去查找一个键并获得该键对应的值：
```go
m := make(map[string]int)
v := m["key1"]
```

乍一看，第二行代码在语法上好像并没有什么不当之处，但其实通过这行语句，无法确定键 key1 是否真实存在于 map 中。\
这是因为，当尝试去获取一个键对应的值的时候，如果这个键在 map 中并不存在，也会得到一个值，这个值是 value 元素类型的零值。

以上面这个代码为例，如果键 key1 在 map 中并不存在，那么 v 的值就会被赋予 value 元素类型 int 的零值，也就是 0，所以这种方式是没有办法正确查找的。

Go 语言的 map 类型支持通过用一种名为 **comma ok** 的惯用法，进行对某个 key 的查询。
```go
m := make(map[string]int)
v, ok := m["key1"]
if !ok {
    // "key1"不在map中
}
// "key1"在map中，v将被赋予"key1"键对应的value
```

可以看到，这里通过一个布尔类型变量 ok，来判断键“key1”是否存在于 map 中。如果存在，变量 v 就会被正确地赋值为键“key1”对应的 value。

不过，如果并不关心某个键对应的 value，而只关心某个键是否在于 map 中，可以使用空标识符 `_` 替代变量 v，忽略可能返回的 value：
```go
m := make(map[string]int)
_, ok := m["key1"]
```

`_, ok` 这种用法在 Go 语言中也非常常见。

### 删除操作

在 Go 语言中，需要借助内置函数 delete 来从 map 中删除数据。

使用 delete 函数的情况下，传入的第一个参数是 map 类型变量，第二个参数就是想要删除的键。
```go
m := map[string]int {
  "key1" : 1,
  "key2" : 2,
}
fmt.Println(m) // map[key1:1 key2:2]
delete(m, "key2") // 删除"key2"
fmt.Println(m) // map[key1:1]
```

### 遍历
和切片一样，使用 `for range` 语句进行遍历。

```go
func doIteration(m map[int]int) {
    fmt.Printf("{ ")
    for k, v := range m {
        fmt.Printf("[%d, %d] ", k, v)
    }
    fmt.Printf("}\n")
}
func main() {
    m := map[int]int{
        1: 11,
        2: 12,
        3: 13,
    }
    for i := 0; i < 3; i++ {
        doIteration(m)
    }
}
```

每次迭代都会返回一个键值对，其中键存在于变量 k 中，它对应的值存储在变量 v 中。

运行上面的示例代码，可能会得到这样的结果：
```
{ [3, 13] [1, 11] [2, 12] }
{ [1, 11] [2, 12] [3, 13] }
{ [3, 13] [1, 11] [2, 12] }
```

这是因为，**对同一 map 做多次遍历的时候，每次遍历元素的次序都不相同。**

## 总结
* map 作为复合类型之一，也是引用类型，没有传递开销
* Go 不允许获取 map 中 value 的地址，这个约束是在编译期间就生效的
* 对同一个 map 多次遍历时，每次元素次序不尽相同，所以不要依赖 map 的元素遍历顺序

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
---
title: Go 语言常见错误和陷阱
date: 2022-12-29 10:30:06
tags: ["Go"]
categories: ["Go"]
---

Go 是一门简单有趣的编程语言，与其他语言一样，在使用时不免会遇到很多坑，不过它们大多不是 Go 本身的设计缺陷。如果你刚从其他语言转到 Go，那这篇文章里的坑多半会踩到。

<!-- more -->

## 初级

### 1.左大括号不能单独放一行

在其他大多数语言中，`{` 的位置你自行决定。Go比较特别，遵守分号注入规则（automatic semicolon injection）：编译器会在每行代码尾部特定分隔符后加;来分隔多条语句，比如会在 ) 后加分号：

```go
// 错误示例
func main()                    
{
    println("hello world")
}

// 等效于
func main();    // 无函数体                    
{
    println("hello world")
}
    ./main.go: missing function body
    ./main.go: syntax error: unexpected semicolon or newline before {

// 正确示例
func main() {
    println("hello world")
}
```

### 2.未使用的变量

如果在函数体代码中有未使用的变量，则无法通过编译，不过全局变量声明但不使用是可以的。即使变量声明后为变量赋值，依旧无法通过编译，需在某处使用它：

```go
// 错误示例
var gvar int     // 全局变量，声明不使用也可以

func main() {
    var one int     // error: one declared and not used
    two := 2    // error: two declared and not used
    var three int    // error: three declared and not used
    three = 3        
}

// 正确示例
// 可以直接注释或移除未使用的变量
func main() {
    var one int
    _ = one

    two := 2
    println(two)

    var three int
    one = three

    var four int
    four = four
}
```

### 3.未使用的 import

如果你 import一个包，但包中的变量、函数、接口和结构体一个都没有用到的话，将编译失败。可以使用 `_`下划线符号作为别名来忽略导入的包，从而避免编译错误，这只会执行 package 的 `init()`

```go
// 错误示例
import (
    "fmt"    // imported and not used: "fmt"
    "log"    // imported and not used: "log"
    "time"    // imported and not used: "time"
)

func main() {
}

// 正确示例
// 可以使用 goimports 工具来注释或移除未使用到的包
import (
    _ "fmt"
    "log"
    "time"
)

func main() {
    _ = log.Println
    _ = time.Now
}
```

### 4.简短声明的变量只能在函数内部使用

```go
// 错误示例
myvar := 1    // syntax error: non-declaration statement outside function body
func main() {
}

// 正确示例
var  myvar = 1
func main() {
}
```

### 5.使用简短声明来重复声明变量

不能用简短声明方式来单独为一个变量重复声明，:=左侧至少有一个新变量，才允许多变量的重复声明：

```go
// 错误示例
func main() {  
    one := 0
    one := 1 // error: no new variables on left side of :=
}

// 正确示例
func main() {
    one := 0
    one, two := 1, 2    // two 是新变量，允许 one 的重复声明。比如 error 处理经常用同名变量 err
    one, two = two, one    // 交换两个变量值的简写
}
```

### 6.不能使用简短声明来设置字段的值

struct 的变量字段不能使用 := 来赋值以使用预定义的变量来避免解决：

```go
// 错误示例
type info struct {
    result int
}

func work() (int, error) {
    return 3, nil
}

func main() {
    var data info
    data.result, err := work()    // error: non-name data.result on left side of :=
    fmt.Printf("info: %+v\n", data)
}

// 正确示例
func main() {
    var data info
    var err error    // err 需要预声明

    data.result, err = work()
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Printf("info: %+v\n", data)
}
```

### 7.不小心覆盖了变量

对从动态语言转过来的开发者来说，简短声明很好用，这可能会让人误会 := 是一个赋值操作符。如果你在新的代码块中像下边这样误用了 :=，编译不会报错，但是变量不会按你的预期工作：

```go
func main() {
    x := 1
    println(x)        // 1
    {
        println(x)    // 1
        x := 2
        println(x)    // 2    // 新的 x 变量的作用域只在代码块内部
    }
    println(x)        // 1
}
```

这是 Go 开发者常犯的错，而且不易被发现。可使用 vet工具来诊断这种变量覆盖，Go 默认不做覆盖检查，添加 -shadow 选项来启用：

```shell
    > go tool vet -shadow main.go
    main.go:9: declaration of "x" shadows declaration at main.go:5
```

注意 vet 不会报告全部被覆盖的变量，可以使用 go-nyet 来做进一步的检测：

```shell
    > $GOPATH/bin/go-nyet main.go
    main.go:10:3:Shadowing variable `x`
```

### 8.显式类型的变量无法使用 nil 来初始化

nil 是 interface、function、pointer、map、slice 和 channel 类型变量的默认初始值。但声明时不指定类型，编译器也无法推断出变量的具体类型。

```go
// 错误示例
func main() {
    var x = nil    // error: use of untyped nil
    _ = x
}

// 正确示例
func main() {
    var x interface{} = nil
    _ = x
}
```

### 9.直接使用值为 nil 的 slice、map

允许对值为 nil 的 slice 添加元素，但对值为 nil 的 map添加元素则会造成运行时 panic

```go
// map 错误示例
func main() {
    var m map[string]int
    m["one"] = 1        // error: panic: assignment to entry in nil map
    // m := make(map[string]int)// map 的正确声明，分配了实际的内存
}    

// slice 正确示例
func main() {
    var s []int
    s = append(s, 1)
}
```

### 10.map 容量

在创建 map 类型的变量时可以指定容量，但不能像 slice 一样使用 cap() 来检测分配空间的大小：

```go
// 错误示例
func main() {
    m := make(map[string]int, 99)
    println(cap(m))     // error: invalid argument m1 (type map[string]int) for cap  
}
```

### 11.string 类型的变量值不能为 nil

对那些喜欢用 nil 初始化字符串的人来说，这就是坑：

```go
// 错误示例
func main() {
    var s string = nil    // cannot use nil as type string in assignment
    if s == nil {    // invalid operation: s == nil (mismatched types string and nil)
        s = "default"
    }
}

// 正确示例
func main() {
    var s string    // 字符串类型的零值是空串 ""
    if s == "" {
        s = "default"
    }
}
```

### 12.Array 类型的值作为函数参数

在 C/C++ 中，数组（名）是指针。将数组作为参数传进函数时，相当于传递了数组内存地址的引用，在函数内部会改变该数组的值。

在 Go 中，数组是值。作为参数传进函数时，传递的是数组的原始值拷贝，此时在函数内部是无法更新该数组的：

```go
// 数组使用值拷贝传参
func main() {
    x := [3]int{1,2,3}

    func(arr [3]int) {
        arr[0] = 7
        fmt.Println(arr)    // [7 2 3]
    }(x)
    fmt.Println(x)            // [1 2 3]    // 并不是你以为的 [7 2 3]
}
```

如果想修改参数数组：

- 直接传递指向这个数组的指针类型：

```go
// 传址会修改原数据
func main() {
    x := [3]int{1,2,3}

    func(arr *[3]int) {
        (*arr)[0] = 7    
        fmt.Println(arr)    // &[7 2 3]
    }(&x)
    fmt.Println(x)    // [7 2 3]
}
```

- 直接使用 slice：即使函数内部得到的是 slice 的值拷贝，但依旧会更新 slice 的原始数据（底层 array）

```go
// 会修改 slice 的底层 array，从而修改 slice
func main() {
    x := []int{1, 2, 3}
    func(arr []int) {
        arr[0] = 7
        fmt.Println(x)    // [7 2 3]
    }(x)
    fmt.Println(x)    // [7 2 3]
}
```

### 13.range 遍历 slice 和 array 时混淆了返回值

与其他编程语言中的 for-in 、foreach 遍历语句不同，Go 中的 range 在遍历时会生成 2 个值，第一个是元素索引，第二个是元素的值：

```go
// 错误示例
func main() {
    x := []string{"a", "b", "c"}
    for v := range x {
        fmt.Println(v)    // 1 2 3
    }
}


// 正确示例
func main() {
    x := []string{"a", "b", "c"}
    for _, v := range x {    // 使用 _ 丢弃索引
        fmt.Println(v)
    }
}
```

### 14.slice 和 array 其实是一维数据

看起来 Go 支持多维的 array 和 slice，可以创建数组的数组、切片的切片，但其实并不是。

对依赖动态计算多维数组值的应用来说，就性能和复杂度而言，用 Go 实现的效果并不理想。

可以使用原始的一维数组、“独立“ 的切片、“共享底层数组”的切片来创建动态的多维数组。

1.使用原始的一维数组：要做好索引检查、溢出检测、以及当数组满时再添加值时要重新做内存分配。

2.使用“独立”的切片分两步：

- 创建外部 slice

  - 对每个内部 slice 进行内存分配

    注意内部的 slice 相互独立，使得任一内部 slice 增缩都不会影响到其他的 slice

```go
// 使用各自独立的 6 个 slice 来创建 [2][3] 的动态多维数组
func main() {
    x := 2
    y := 4

    table := make([][]int, x)
    for i  := range table {
        table[i] = make([]int, y)
    }
}
```

1.使用“共享底层数组”的切片

- 创建一个存放原始数据的容器 slice
- 创建其他的 slice
- 切割原始 slice 来初始化其他的 slice

```go
func main() {
    h, w := 2, 4
    raw := make([]int, h*w)

    for i := range raw {
        raw[i] = i
    }

    // 初始化原始 slice
    fmt.Println(raw, &raw[4])    // [0 1 2 3 4 5 6 7] 0xc420012120 

    table := make([][]int, h)
    for i := range table {

        // 等间距切割原始 slice，创建动态多维数组 table
        // 0: raw[0*4: 0*4 + 4]
        // 1: raw[1*4: 1*4 + 4]
        table[i] = raw[i*w : i*w + w]
    }

    fmt.Println(table, &table[1][0])    // [[0 1 2 3] [4 5 6 7]] 0xc420012120
}
```

更多关于多维数组的参考

* [go-how-is-two-dimensional-arrays-memory-representation](https://stackoverflow.com/questions/39561140/how-is-two-dimensional-arrays-memory-representation)

* [what-is-a-concise-way-to-create-a-2d-slice-in-go](https://stackoverflow.com/questions/39804861/what-is-a-concise-way-to-create-a-2d-slice-in-go)

### 15.访问 map 中不存在的 key

和其他编程语言类似，如果访问了 map 中不存在的 key 则希望能返回 nil，比如在 PHP 中：

```shell
    > php -r '$v = ["x"=>1, "y"=>2]; @var_dump($v["z"]);'
    NULL
```

Go 则会返回元素对应数据类型的零值，比如 nil、'' 、false 和 0，取值操作总有值返回，故不能通过取出来的值来判断 key 是不是在 map 中。

通常使用 comma ok 惯用法来判断 key 是否存在：

```go
// 错误的 key 检测方式
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if v := x["two"]; v == "" {
        fmt.Println("key two is no entry")    // 键 two 存不存在都会返回的空字符串
    }
}

// 正确示例
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if _, ok := x["two"]; !ok {
        fmt.Println("key two is no entry")
    }
}
```

### 16.string 类型的值是常量，不可更改

尝试使用索引遍历字符串，来更新字符串中的个别字符，是不允许的。

string 类型的值是只读的二进制 byte slice，如果真要修改字符串中的字符，将 string 转为 []byte 修改后，再转为 string 即可：

```go
// 修改字符串的错误示例
func main() {
    x := "text"
    x[0] = "T"        // error: cannot assign to x[0]
    fmt.Println(x)
}

// 修改示例
func main() {
    x := "text"
    xBytes := []byte(x)
    xBytes[0] = 'T'    // 注意此时的 T 是 rune 类型
    x = string(xBytes)
    fmt.Println(x)    // Text
}
```

注意： 上边的示例并不是更新字符串的正确姿势，因为一个 UTF8 编码的字符可能会占多个字节，比如汉字就需要 `3~4`个字节来存储，此时更新其中的一个字节是错误的。

更新字串的正确姿势：将 string 转为 rune slice（此时 1 个 rune 可能占多个 byte），直接更新 rune 中的字符

```go
func main() {
    x := "text"
    xRunes := []rune(x)
    xRunes[0] = '我'
    x = string(xRunes)
    fmt.Println(x)    // 我ext
}
```

### 17.string 与 byte slice 之间的转换

当进行 string 和 byte slice 相互转换时，参与转换的是拷贝的原始值。这种转换的过程，与其他编程语的强制类型转换操作不同，也和新 slice 与旧 slice 共享底层数组不同。

Go 在 string 与 byte slice 相互转换上优化了两点，避免了额外的内存分配：

- 在 map[string] 中查找 key 时，使用了对应的 []byte，避免做 m[string(key)] 的内存分配
- 使用 for range 迭代 string 转换为 []byte 的迭代：for i,v := range []byte(str) {...}

### 18.string 与索引操作符

对字符串用索引访问返回的不是字符，而是一个 byte 值。

这种处理方式和其他语言一样，比如 PHP 中：

```shell
> php -r '$name="中文"; var_dump($name);'    # "中文" 占用 6 个字节
string(6) "中文"

> php -r '$name="中文"; var_dump($name[0]);' # 把第一个字节当做 Unicode 字符读取，显示 U+FFFD
string(1) "�"    

> php -r '$name="中文"; var_dump($name[0].$name[1].$name[2]);'
string(3) "中"
func main() {
    x := "ascii"
    fmt.Println(x[0])        // 97
    fmt.Printf("%T\n", x[0])// uint8
}
```

如果需要使用 `for range` 迭代访问字符串中的字符（`unicode code point / rune`），标准库中有 `"unicode/utf8"` 包来做 `UTF8` 的相关解码编码。另外 `utf8string` 也有像 `func (s *String) At(i int) rune` 等很方便的库函数。

### 19.字符串并不都是 UTF8 文本

string 的值不必是 UTF8 文本，可以包含任意的值。只有字符串是文字字面值时才是 UTF8 文本，字串可以通过转义来包含其他数据。

判断字符串是否是 UTF8 文本，可使用 "unicode/utf8" 包中的 ValidString() 函数：

```go
func main() {
    str1 := "ABC"
    fmt.Println(utf8.ValidString(str1))    // true

    str2 := "A\xfeC"
    fmt.Println(utf8.ValidString(str2))    // false

    str3 := "A\\xfeC"
    fmt.Println(utf8.ValidString(str3))    // true    // 把转义字符转义成字面值
}
```

### 20.字符串的长度

在 Python 中：

```
    data = u'♥'  
    print(len(data)) # 1
```

然而在 Go 中：

```go
func main() {
    char := "♥"
    fmt.Println(len(char))    // 3
}
```

Go 的内建函数 len() 返回的是字符串的 byte 数量，而不是像 Python 中那样是计算 Unicode 字符数。

如果要得到字符串的字符数，可使用 "unicode/utf8" 包中的 RuneCountInString(str string) (n int)

```go
func main() {
    char := "♥"
    fmt.Println(utf8.RuneCountInString(char))    // 1
}
```

注意： RuneCountInString 并不总是返回我们看到的字符数，因为有的字符会占用 2 个 rune：

```go
func main() {
    char := "é"
    fmt.Println(len(char))    // 3
    fmt.Println(utf8.RuneCountInString(char))    // 2
    fmt.Println("cafe\u0301")    // café    // 法文的 cafe，实际上是两个 rune 的组合
}
```

## 中级

## 参考链接
* [Golang新手可能会踩的58个坑](https://www.topgoer.com/%E8%B5%84%E6%96%99%E4%B8%8B%E8%BD%BD/Golang%E6%96%B0%E6%89%8B%E5%8F%AF%E8%83%BD%E4%BC%9A%E8%B8%A9%E7%9A%8450%E4%B8%AA%E5%9D%91.html?h=%E5%9D%91)
* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)
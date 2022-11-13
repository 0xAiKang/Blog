---
title: Go 语言学习笔记——方法接收器的选择
date: 2022-10-31 22:34:23
tags: ["Go"]
categories: ["Go"]
---

上一篇笔记了解了 Go 语言方法的声明、本质，这篇笔记主要来了解 Go 语言方法的receiver 参数。

## receiver 参数类型对 Go 方法的影响

要想为 receiver 参数选出合理的类型，需要先要了解不同的 receiver 参数类型会对 Go 方法产生怎样的影响。

因为**方法的本质就是函数**，所以下面从等价转换之后的函数的角度来分析一下对函数有什么影响，间接得出它对 Go 方法的影响呢。

还是从一段示例代码开始。
```go
package main
  
type T struct {
    a int
}
func (t T) M1() {
    t.a = 10
}
func (t *T) M2() {
    t.a = 11
}
func main() {
    var t T
    println(t.a) // 0
    t.M1()
    println(t.a) // 0
    p := &t
    p.M2()
    println(t.a) // 11
}
```

在上面的示例中，为基类型分别定义了两个方法 M1 和 M2，其中receiver 参数类型分别是 `T` 和 `*T`，最后两个方法都通过参数 t 对 t的成员进行了修改。

通过运行示例代码之后，可以看到，方法 M1 对成员的修改并没有成功，还是原值，而方法 M2 对成员的修改成功了。因此可以得出以下结论：

* 当 receiver 参数的类型为 `T`（非指针）时：receiver 参数实际上是**T 类型实例的副本**，因此对参数 t 进行任何修改都**不会影响**到原实例。
* 当 receiver 参数的类型为 `*T`（指针）时：receiver 参数实际上是**T 类型实例的地址**，因此对参数 t 进行任何修改都**会影响**到原实例。

了解了不同类型的 receiver 参数对 Go 方法的影响后，就可以总结一下，日常编码中选择 receiver 的参数类型的时候，我们可以参考哪些原则。

## 原则一
**如果 Go 方法要把对 receiver 参数代表的类型实例的修改，反映到原类型实例上，那么我们应该选择 `*T` 作为 receiver 参数的类型**。

这个原则很好理解，依据实际情况选择合适的。

不过这个时候可能会有一个问题：\
选择`*T` 作为 receiver 参数的类型，那么是不是只能通过 `*T` 类型的实例调用该方法，而不能通过 `T` 类型的实例调用？

正好这也是上一篇笔记中，遗留下了一个问题。

将上面的示例代码改造一下：
```go
type T struct {
      a int
  }
  
  func (t T) M1() {
      t.a = 10
  }
 
 func (t *T) M2() {
     t.a = 11
 }
 
 func main() {
     var t1 T
     println(t1.a) // 0
     t1.M1()
     println(t1.a) // 0
     t1.M2()
     println(t1.a) // 11
 
     var t2 = &T{}
     println(t2.a) // 0
     t2.M1()
     println(t2.a) // 0
     t2.M2()
     println(t2.a) // 11
 }
```

运行示例代码查看输出结果，会发现类型为 `T` 的实例 t1，不仅可以调用 receiver 参数类型为 `T` 的方法 M1，它还可以直接调用 receiver 参数类型为 `*T` 的方法 M2，并且调用完 M2 方法后，成员的值也被修改了。

直接说结论：这是因为 Go 编译器，在背后帮我们做了自动转换。

或者说，`t1.M2()` 这种用法是 Go 提供的“语法糖”：Go 判断 t1 的类型为 `T`（非指针），也就是与方法 M2 的 receiver 参数类型 `*T`（指针） 不一致后，会自动将 `t1.M2()` 转换为 `(&t1).M2()`。

同理，`t2.M1()` 这种用法也是因为 Go 编译器在背后做了转换。也就是，Go 判断 t2 的类型为 `*T`（指针），与方法 M1 的 receiver 参数类型 T（非指针）不一致，就会自动将 `t2.M1()` 转换为`(*t2).M1()`。

结论：**无论是 `T` 类型实例，还是 `*T` 类型实例，都既可以调用 receiver 为 `T`类型的方法，也可以调用 receiver 为 `*T` 类型的方法。**

这里做了两次自动转换，涉及到了指针的运用，如果不理解，可以看下这篇笔记。

---

对于实例 t1，已经是一个变量，所以对于它而言，只有取址符可以用。（因为只有变量地址才能使用 * 操作符）

对于实例 t1 而言，它只是一个变量，类型为 `T`，所以只能进行取址操作。

对于实例 t2 而言，它是一个变量地址，类型为 `*T`，所以也可以说是 `*T` 的指针类型。
所以可以通过 `*t2` 进行取值操作。

而类型 T，它是一个结构体，所以它既可以使用& 获取自己的地址，也可以使用 * 作为一个结构体指针。

语法糖，自动转换。

## 原则二
第一个原则说的是，当要在方法中对 receiver 参数代表的类型实例进行修改，那要为 receiver 参数选择 `*T` 类型，但是如果不需要在方法中对类型实例进行修改呢？\
这个时候是选择 `T` 类型还是 `*T` 类型呢？

通常会为 receiver 参数选择 `T` 类型，这是因为可以缩窄外部修改类型实例内部状态的“接触面”，也就是尽量少暴露可以修改类型内部状态的方法。

不过也有一个例外需要你特别注意。考虑到 Go 方法调用时，receiver 参数是以值拷贝的形式传入方法中的。那么，如果 receiver 参数类型的 size 较大，以值拷贝形式传入就会导致较大的性能开销，这时我们选择 *T 作为 receiver 类型可能更好些。

以上这些可以作为我们选择 receiver 参数类型的第二个原则。

到这里，可能会觉得，前两个原则似乎并不难理解，这是因为这两条只是基础原则，还有一条比较难的原则在下面。

不过在讲解这第三条原则之前，需要先要了解一个基本概念：**方法集合（Method Set）**，它是理解第三条原则的前提。

## 方法集合
在了解方法集合是什么之前，我们先通过一个示例，直观了解一下为什么要有方法集合，它主要用来解决什么问题：

```go
type Interface interface {
    M1()
    M2()
}
type T struct{}
func (t T) M1()  {}
func (t *T) M2() {}
func main() {
    var t T
    var pt *T
    var i Interface
    i = pt
    i = t 
}
```

上面的示例代码定义了一个接口类型 Interface 以及一个自定义类型 T。Interface 接口类型包含了两个方法 M1 和 M2，它们的基类型都是 T，但它们的 receiver 参数类型不同，一个为  `T`，另一个为 `*T`。\
在 main 函数中，分别将 `T` 类型实例 t 和 `*T` 类型实例 pt 赋值给 Interface 类型变量 i。

运行示例代码，会发现编译失败了：

> cannot use t (type T) as type Interface in assignment: T does not implement Interface (M2 method has pointer receiver)
>

大意是：T 没有实现 Interface 类型方法列表中的 M2，因此类型 T 的实例 t 不能赋值给 Interface 变量

在解决这个问题之前，先来了解一下什么是方法集合。

Go 中任何一个类型都有属于自己的方法，或者说方法集合是 Go 类型的一个“属性”。但是不是所有类型都有自己的方法，比如 int 类型就没有，所以，对于没有定义方法的Go 类型，称其拥有空方法集合。

接口类型类型相对特殊，它只会列出代表接口的方法列表，不会具体定义某个方法，它的方法集合就是它的方法列表中的所有方法，因为下面重点讲解的是非接口类型的方法集合。

为了方便查看一个非接口类型的方法集合，提供了一个函数 dumpMethodSet，用于输出一个非接口类型的方法集合：
```go
func dumpMethodSet(i interface{}) {
    dynTyp := reflect.TypeOf(i)
    if dynTyp == nil {
        fmt.Printf("there is no dynamic type\n")
        return
    }
    n := dynTyp.NumMethod()
    if n == 0 {
        fmt.Printf("%s's method set is empty!\n", dynTyp)
        return
    }
    fmt.Printf("%s's method set:\n", dynTyp)
    for j := 0; j < n; j++ {
        fmt.Println("-", dynTyp.Method(j).Name)
    }
    fmt.Printf("\n")
}
```

下面则利用这个函数，试着输出一下 Go 原生类型以及自定义类型的方法集合：
```go
type T struct{}

func (T) M1() {}
func (T) M2() {}

func (*T) M3() {}
func (*T) M4() {}

func main() {
    var n int
    dumpMethodSet(n)
    dumpMethodSet(&n)

  var t T
    dumpMethodSet(t)
    dumpMethodSet(&t)
}
```

运行上面的示例代码，得到如下输出：
```go
int's method set is empty!
*int's method set is empty!
main.T's method set:
- M1
- M2

*main.T's method set:
- M1
- M2
- M3
- M4
```

从上面的输出中，可以看到`int`、`*int` 是 Go 原生类型由于没有定义方法，所以它们的方法集合都是空的。\
而自定义类型 T 定义了方法 M1 和 M2，因此它的方法集合包含了 M1 和 M2，符合预期，但是 `*T` 的方法集合除了预期的 M3 和 M4 之外，怎么还包含了类型 T 的 M1 和 M2 方法？

这是因为，Go 语言规定，**`*T` 类型的方法集合包含所有以 `*T` 为 receiver 参数类型的方法，以及所有以 `T` 为 receiver 参数类型的方法。**\
这就是这个示例中为何 `*T` 类型的方法集合包含四个方法的原因。

这个时候再来看看前面的那个编译失败的问题，是不是就找到原因了。

可以使用 `dumpMethodSet` 函数，输出一下该示例中t 与 pt 各自所属类型的方法集合：
```go
main.T's method set:
- M1
*main.T's method set:
- M1
- M2
```

从输出结果中，可以看到 `T`、`*T` 各自的方法集合确实是符合上面的结论的。

到这里，已经知道了所谓的**方法集合决定接口实现**的含义就是：如果某类型 `T` 的方法集合与某接口类型的方法集合相同，或者类型 `T` 的方法集合是接口类型 `I` 方法集合的超集，那么我们就说这个类型 `T` 实现了接口 `I`。或者说，方法集合这个概念在 Go 语言中的主要用途，就是用来**判断某个类型是否实现了某个接口**。

有了方法集合的概念做铺垫，选择 receiver 参数类型的第三个原则也相对好理解了。

## 原则三
该原则的选择依据就是 **`T` 类型是否需要实现某个接口**。

如果 **`T` 类型需要实现某个接口的全部方法，那就要使用 `T` 作为 receiver 参数的类型，来满足接口类型方法集合中的所有方法**。

如果 **`T` 类型不需要实现某一接口的全部方法，那么参考原则一和原则二就可以了**。

## 总结
* 当 receiver 参数的类型为 `T` 时，对receiver 参数的任何修改都**不会**影响到原实例
* 当 receiver 参数的类型为 `*T` 时，对receiver 参数的任何修改都**会**影响到原实例
* 实际进行 Go 方法设计时，首先应该考虑原则三，其次才是原则一和原则二
* 方法集合是用来**判断某一类型是否实现了某接口的唯一手段**

## 参考链接
* [Tony Bai · Go 语言第一课](https://time.geekbang.org/column/intro/100093501)
---
title: Go 语言学习笔记——结构体标签
date: 2022-11-06 22:47:05
tags: ["Go"]
categories: ["Go"]
---

结构体标签就是对结构体字段的额外信息补充。

<!-- more -->

## 定义结构体标签

结构体标签的定义方式是，在字段声明后面可以跟一个可选的字符串文字。
```go
type T struct {
    f1     string "f one"
    f2     string
    f3     string `f three`
    f4, f5 int64  `f four and five`
  	f6     string `key1:"value1" key1:"value2"`
	f7     string "json:\"f7\""   // 可读性很低
}
```
通常使用 \` 反引号来定义结构体标签，例如上面结构体 T 的 f6 字段。

为什么说是通常呢？\
其实也可以使用双引号去定义，但是遇到需要转义时，就比较麻烦了，开发者总是需要去关心如何转义的问题，而使用反引号则完全不用担心这个问题。

因此绝大多数情况都是反引号去定义的。

## 从结构体标签获取值
上面提到了，结构体字段只需要在字段声明后面跟上字符串就行，但是大多数情况下，不会跟一个毫无规则的字符串，因为这样没有意义。

往往会这样定义一个结构体标签：**由一个或多个键值对组成。键与值使用冒号分隔，值用双引号括起来。键值对之间使用一个空格分隔**，例如：`key1:"value1" key2:"value2"`。

标签可通过 reflect（反射） 包访问，因为这些信息是静态的，因此不需要实例化结构体，就能直接获取到。

结构体标签中常用的一些方法：
* `Get`：根据 Tag 中的键获取对应的值
* `Lookup`：根据 Tag 中的键，查询值是否存在

```go
type T struct {
  f1     string "f one"
	f2     string
	f3     string `f three`
	f4, f5 int64  `f four and five`
	f6     string `key1: "value1" key1:"value2"`
	f7     string "json:\"f7\""
}

func main() {
	t := reflect.TypeOf(T{})

	f1, _ := t.FieldByName("f1")
	fmt.Println(f1.Tag)           // f one
	fmt.Println(f1.Tag.Get("f1")) //

	f2, _ := t.FieldByName("f2")
	fmt.Println(f2.Tag)         //
	fmt.Println(f2.Tag.Get("")) //

	f3, _ := t.FieldByName("f3")
	fmt.Println(f3.Tag)           // f three
	fmt.Println(f3.Tag.Get("f1")) //

	f4, _ := t.FieldByName("f4")
	fmt.Println(f4.Tag)           // f four and five
	fmt.Println(f4.Tag.Get("f4")) //

	f5, _ := t.FieldByName("f5")
	fmt.Println(f5.Tag)           // f four and five
	fmt.Println(f5.Tag.Get("f5")) // 
  
  // 可以看到上面部分的实例代码，通过结构体方法获取不到有效的值
  // 这也是为什么不会直接跟一个字符串的原因，因为结构体标签的方法在这种情况下作用体现不出来

	f6, _ := t.FieldByName("f6")
	fmt.Println(f6.Tag)                // key1:"value1" key1:"value2"
	fmt.Println(f6.Tag.Get("key1"))    // value1
	fmt.Println(f6.Tag.Lookup("key1")) // value1 true

	f7, _ := t.FieldByName("f7")
	fmt.Println(f7.Tag)                // json:"f7"
	fmt.Println(f7.Tag.Get("json"))    // f7
	fmt.Println(f7.Tag.Lookup("json")) // f7 true
}
```

## 一个细节

需要注意 💡 的是：编写 Tag 时，必须严格遵守键值对的规则。

结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，例如下面这个例子：

```go
type cat struct {
	Name string
	Type int `json: "type" id:"100"`
}

func main() {
	typeOfCat := reflect.TypeOf(cat{})

	if catType, ok := typeOfCat.FieldByName("Type"); ok {
		fmt.Println(catType.Tag.Get("json"))
	}
}
```
上面的实例代码，一眼看上去并没有什么问题是吧？

但是实际运行并不会输出期望的 type，这是因为在结构体标签这一行，在json:和"type"之间增加了一个空格。这种写法没有遵守结构体标签的规则，因此无法通过 `Tag.Get` 获取到正确的 json 对应的值。

这个错误在开发中非常容易被疏忽，造成难以察觉的错误。

---

结构体标签有许多应用场景，例如配置管理，结构的默认值，验证，命令行参数描述等，可以从[这个列表](https://github.com/golang/go/wiki/Well-known-struct-tags#list-of-well-known-struct-tags)中查看到有哪些项目使用了结构体标签。
---
title: Go 语言学习——json 结构体标签
date: 2022-12-10 17:36:04
tags: ["Go"]
categories: ["Go"]
---

使用 Golang 时，经常需要创建结构体，而结构体经常需要被序列化或者反序列化，通常会在结构体的 field 类型后加上 json 结构体标签。

<!-- more -->

json 结构体标签，常见的关键字有 `omitempty` 和 `-`，下面一起来看一下，这几种的区别。

示例代码如下：
```go
type User struct {
	Id       string
	Name     string
	age      int
	gender   string
	nickname string
}

type User2 struct {
	Id       string `json:"id"`
	Name     string `json:"name"`
	Age      int    `json:"age"`
	Gender   string `json:"gender"`
	NickName string `json:"nickname"`
}

type User3 struct {
	Id       string `json:"id"`
	Name     string `json:"name,omitempty"`
	Age      int    `json:"age,omitempty"`
	Gender   string `json:"gender,omitempty"`
	NickName string `json:"nickname,omitempty"`
}

type User4 struct {
	Id       string `json:"id"`
	Name     string `json:"name,omitempty"`
	Age      int    `json:"-"`
	Gender   string `json:"gender,omitempty"`
	NickName string `json:"nickname,omitempty"`
}

func main() {
	u := User{
		Id:     "1",
		Name:   "张三",
		age:    20,
		gender: "男",
	}
	data, err := json.Marshal(u)
	if err != nil {
		fmt.Println(err.Error())
	}

	u2 := User2{
		"1",
		"张三",
		20,
		"男",
		"",
	}
	data2, _ := json.Marshal(u2)

	u3 := User3{
		Id: "1",
	}
	data3, _ := json.Marshal(u3)

	u4 := User2{
		Id: "1",
	}
	data4, _ := json.Marshal(u4)

	u5 := User4{
		Id:     "1",
		Name:   "张三",
		Age:    19,
		Gender: "男",
	}
	data5, _ := json.Marshal(u5)
	fmt.Printf("%s ：只会打印出首字母大写的 field，首字母小写的 field 只允许在包内使用，没有使用 json 结构体标签，序列化之后的 json 数据，field 不会发生变化 \n", string(data))
	fmt.Printf("%s :使用 json 结构体标签，序列化之后的 json 数据，field 和 json 结构体保持一致 \n", string(data2))
	fmt.Printf("%s :使用 json 结构体标签，同时使用 omitempty 标记，初始化时，只有 Id 字段进行赋值 \n", string(data3))
	fmt.Printf("%s :使用 json 结构体标签，未使用 omitempty 标记，初始化时，只有 Id 字段进行赋值 \n", string(data4))
	fmt.Printf("%s :使用 json 结构体标签，同时使用 - 标记以及 omitempty 标记 \n", string(data5))
}
```

运行上面的示例代码，得到以下输出：
```json
{"Id":"1","Name":"张三"} ：只会打印出首字母大写的 field，首字母小写的 field 只允许在包内使用，没有使用 json 结构体标签，序列化之后的 json 数据，field 不会发生变化 
{"id":"1","name":"张三","age":20,"gender":"男","nickname":""} :使用 json 结构体标签，序列化之后的 json 数据，field 和 json 结构体保持一致 
{"id":"1"} :使用 json 结构体标签，同时使用 omitempty 标记，初始化时，只有 Id 字段进行赋值 
{"id":"1","name":"","age":0,"gender":"","nickname":""} :使用 json 结构体标签，未使用 omitempty 标记，初始化时，只有 Id 字段进行赋值 
{"id":"1","name":"张三","gender":"男"} :使用 json 结构体标签，同时使用 - 标记以及 omitempty 标记
```

## 总结
* json 结构体标签，用于结构体序列化时，将结构体与 json 数据解藕
* 定义为 `json:"field"` 格式的结构体，在初始化时，field 字段不能省略，否则会编译错误
* 定义为 `json:"nickname,omitempty"` 格式的结构体，在初始化时，field 字段可以省略。当省略 field 字段进行赋值时，序列化之后的 json 也不会包含该字段，反之则会包含
* 定义为 `json:"-"` 格式的结构体，初始化时，无论是否赋值，序列化之后的 json 都不会包含该字段
---
title: 如何将 JSON 对象转换成 PHP 数组
date: 2020-09-22 22:58:35
tags: ["PHP"]
categories: ["PHP"]
---

在介绍如何将JSON 字符串转传为PHP 数组之前，先来复习一下什么是JSON。

<!-- more -->

### JSON
通俗一点讲JSON 就是一种数据结构，就是一串字符串，只不过元素会通过特定的符号标注。

* `{}`：大括号表示对象
* `[]`：中括号表示数组
* `""`：双引号内是属性或值

标准的JSON 对象：
```
# 这是一个JSON 对象
{
    "name": "boo",
    "gender": "men",
    "age": 25
}
```

标准的JSON 数组：
```
# 这是一个包含两个对象的JSON 数组
[   
    {
        "name": "boo",
        "gender": "men",
        "age": 25
    },
    {
        "name": "max",
        "gender": "men",,
        "age": 29
    }
]

# 这是一个包含数组的JSON 对象
{
    "name":["Michael","Jerry"]
}
```

在熟悉了几种常见的JSON 字符串之后，在来看一下如何解析JSON 字符串。
### json_decode
JSON 对象转换为对象
```
<?php
$jsonObj = '{"name": "boo"}';

$obj = json_decode($jsonObj);
print $obj->{"name"};   //boo

$jsonObj2 = '[{"name": "boo"}]';

$obj2 = json_decode($jsonObj2);
print $obj[0]->{"name"};   //boo

object(stdClass)[1]
      public 'name' => string 'boo' (length=3)

array (size=1)
  0 => 
    object(stdClass)[1]
      public 'name' => string 'boo' (length=3)
      
```

JSON 对象转换为数组
```
<?php
$jsonObj = '{"name": "boo"}';

$arr = json_decode($jsonObj, true);
print $arr['name'];     //boo

array (size=1)
      'name' => string 'boo' (length=3)
```

需要注意几个容易出错的细节：
```
<?php
// 大括号外需要使用单引号
$bad_json = "{ 'bar': 'baz' }";
json_decode($bad_json);    // null

// 属性需要使用双引号引起来
$bad_json = '{ bar: "baz" }';
json_decode($bad_json);    // null

// 不允许尾随逗号
$bad_json = '{ bar: "baz", }';
json_decode($bad_json);    // null

```

### json_encode
下面来看看如何返回JSON 格式的数据，通过使用 `json_encode`这个函数：
```
$b = array();

echo "空数组作为数组输出: ", json_encode($b), "\n";  //空数组作为数组输出：[]
echo "空数组作为对象输出: ", json_encode($b, JSON_FORCE_OBJECT), "\n\n";   //空数组作为对象输出：{}

$c = array(array(1,2,3));

echo "多维数组作为数组输出: ", json_encode($c), "\n";       //多维数组作为数组输出：[[1,2,3]]
echo "多维数组作为对象输出: ", json_encode($c, JSON_FORCE_OBJECT), "\n\n";      //多维数组作为对象输出：{"0":{"0":1,"1":2,"2":3}}

$d = array('foo' => 'bar', 'baz' => 'long');

echo "关联数组只能作为对象输出: ", json_encode($d), "\n";       //关联数组只能作为对象输出：{"foo":"bar","baz":"long"}
echo "关联数组只能作为对象输出: ", json_encode($d, JSON_FORCE_OBJECT), "\n\n";      //关联数组只能作为对象输出：{"foo":"bar","baz":"long"}

$arr = array(
  "name" => "boo",
  "gender" => "men",
  "age" => 22
);

$res = json_encode($arr);
var_dump($res);
echo($res);

string '{"name":"boo","gender":"men","age":22}' (length=35)
{"name":"boo","gender":"men","age":22}
```
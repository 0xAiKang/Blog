---
title: MysqliDb 快速上手
date: 2020-10-02 15:26:03
tags: ["Mysql", "Mysqli"]
categories: ["Mysql"]
---

MysqliDb 是基于 mysqli 扩展出来的一个类库，其中封装了很多常用的Mysql 基础操作，相比原生的方式，后者使用起来更加方便。

具有如下特点：
1. 支持链式操作
2. 支持Mysql 函数的使用
3. ...

## 安装
使用composer 安装
```
composer require thingengineer/mysqli-database-class:dev-master
```
因为`MysqliDb`没有命名空间，所以我们想要使用的话，不能自动加载，只能先引入。

```
require "MysqliDb.php";
```

### 初始化
初始化连接有几种方式：

#### 1. MysqliDb 字符串
```
$db = new MysqliDb ('host', 'username', 'password', 'databaseName');
```

#### 2. MysqliDb 对象
```
$db = new MysqliDb ([
  'host' => 'host',
  'username' => 'username', 
  'password' => 'password',
  'db'=> 'databaseName',
  'port' => 3306,
  'prefix' => 'my_',
  'charset' => 'utf8'
]);
```

#### 3. mysqli 对象

```
$mysqli = new mysqli ('host', 'username', 'password', 'databaseName');
$db = new MysqliDb ($mysqli);
```

### 新增
向user 表中插入一条记录：
```
$data = [
  "name" => "boo",
  "age" => 21,
  "gender" => "man"
];
$success = $db->insert("user", $data);
```
返回值类型：bool

### 修改
修改user 表中的一条记录
```
$data = [
  "age" => 22,
];
$success = $db->where(["name" => "boo"])
   ->update("user", $data);
```
返回值类型：bool

### 查询
#### 获取user 表所有数据：
```
$result = $db->get("user", null, "*");
```
返回值：多维数组

#### 获取user 表单条数据：
```
$result = $db->getOne("user",  "*");
```
返回值：关联数组

#### 获取user 表单个字段的值：
```
$result = $db->where("name", "boo")
	 ->getValue("user", "*");
```
返回值：string

#### 获取查询条数：
```
$result = $db->getValue("user", "count(*)");
```

### 删除
删除user 表中一条记录
```
$success = $db->where("user_id", "boo")
  ->delete("user);
```

### 运行原生SQL

```
$result = $db->rawQuery("select * from user where name = \"boo\"")
```

总体来说，MysqliDb 真的挺好用的，基本上可以满足所有日常需求。
这里只是列举了最基本的CURD，更多操作可以参考[官网手册](https://github.com/joshcam/PHP-MySQLi-Database-Class)。
### 参考链接
[joshcam/mysqli-database-class](https://packagist.org/packages/joshcam/mysqli-database-class)


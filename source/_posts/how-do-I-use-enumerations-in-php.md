---
title: 如何在 PHP 中使用枚举
date: 2021-03-08 20:57:51
tags: ["PHP"]
categories: ["PHP"]
---

在编写业务代码时，常常会遇到状态或者类型不一致造成的逻辑分支，这时，最忌讳的是直接在业务代码中对数值进行判断。

<!-- more -->

那么更好的方式应该是怎样呢？**使用枚举**。

使用枚举有以下几个好处：
1. 减少因为直接输入数字而导致的错误
2. 使代码更易于阅读
3. 方便维护，后面需要添加新的类型时，不会突兀

PHP 本身不支持枚举，但是使用类中的常量去定义可以实现等价的效果。

## 定义枚举
下面为用户类型创建一个枚举，用户可以是以下三种类型之一：
1. 普通用户
2. 管理员
3. 超级管理员

看起来像这样：
```
class UserType extends Enum
{
  const MEMBER = 1;
  const ADMIN = 2;
  const SUPERADMIN = 3;
}
```

### 使用枚举

```
if ($user->type === UserType::MEMBER){
  // todo 
}
```

如果我们不使用枚举，代码可能就会变成这样：
```
if ($user->type === 1) { // 这个1表示什么??
    // todo
}

if ($user->type === 'Member') { // 这他妈咋么又是字符串 😞
    // todo 
}
```

## 定义获取器
很多时候我们希望能获取到某个类型对应的具体含义，这时可以通过定义获取器来获取。

```
class UserType extends Enum
{
  public static $userType = [
  	self::MEMBER => "普通会员",
  	self::ADMIN => "管理员",
  	self::SUPERADMIN => "超级管理员"
  ];
  
  public static function getUserType($type)
  {
  	return self::$voucherMap[$type];
  } 
}
```
这样当我们在调用`getUserType` 方法时，只需要传入对应的类型，就能获取到`普通会员`、`管理员`、`超级管理员`了。

## 参考链接
* [在 Laravel 中使用枚举](https://learnku.com/laravel/t/36091)
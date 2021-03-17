---
title: 什么是 N+1 问题，以及如何解决
date: 2021-03-15 22:10:11
tags: ["PHP", "Laravel", "ThinkPHP"]
categories: ["PHP"]
---

`N+1` 是ORM（对象关系映射）关联数据读取中存在的一个问题。

<!-- more -->

在介绍什么是`N+1`问题之前，首先思考一个问题：

假设现在有一个用户表（User）和一个余额表（Balance），这两个表通过`user_id`进行关联。现在有一个需求是**查询年龄大于18岁的用户，以及用户各自的余额**。

这个问题并不难，但对于新手而言，可能常常会犯的一个错误就是在循环中进行查询。

```
$users = User::where("age", ">", 18)->select();
foreach($users as $user){
  $balance = User::getFieldByUserId($user->user_id, "balance");
  $user['balance'] = $balance;
}
```
这样做是非常糟糕的，数据量小还少，在数据量较大的情况下，是非常消耗数据库性能的。

通过Mysql 查询日志，可以看到查询用户表是一次，因为有四个符合该条件的用户，查询用户表关联的余额表是四次。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210314132050.png)

`N+1`问题就是这样产生的：查询主表是一次，查询出N 条记录，根据这N 条记录，查询关联的副（从）表，共需要N 次。所以，应该叫`1+N` 问题更合适一些。

其实，如果稍微了解一点SQL，根本不用这么麻烦，直接使用`JOIN` 一次就搞定了。

对于这类问题，ORM 其实为我们提供了相应的方案，那就是使用『预加载功能』。

### 预加载功能

使用`with()`方法指定想要预加载的关联：
```
$users = User::where("age", ">", 18)
		->with("hasBalance")
		->select();
```

`hasBalance` 是什么呢？

它是在`User`模型中定义的一个方法：
```
class User extends Model
{
    //  ...
    
    // User模型与Balance 模型进行一对一关联
    public function hasBalance()
    {
    	  return $this->hasOne(Balance::class, "user_id", "user_id");
    }
}
```

通过这个方法让`User` 模型与`Balance` 模型进行一对一关联。

现在再来看一下Mysql 的查询日志：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210314133535.png)

可以很清楚的看到，总查询次数由原来的`1+N` 变成了现在的`1+1`。

## 总结
`N+1` 问题是什么？会造成什么影响？应该如何解决？
1. 执行一次查询获取N 条主数据后，由于关联引起的执行N 次查询从数据
2. 带来了不必要的查询开销
3. 可以通过框架 ORM 自带的`with` 去解决
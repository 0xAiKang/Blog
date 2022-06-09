---
title: Eloquent ORM 常见使用场景整理
date: 2022-06-09 22:57:36
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

以下实例，都是基于 Eloquent ORM，可以直接在 Laravel 下直接使用。

<!-- more -->

## 场景一
用户表增加了一个字段，因为项目中有许多指定部分列的查询，现在希望项目中所有查询用户信息的地方都可以自动加上这个字段。

解决方案：重写用户模型 booted 方法，并闭包自定义全局查询作用域。
```php
protected static function booted()
{
    static::addGlobalScope("uid", function (Builder $builder) {
        $columns = $builder->getQuery()->columns;
        if (!is_null($columns)) {
            $builder->addSelect(["is_vip",]);
        }
    });
}
```

这样，当执行以下查询时，生成 SQL 如下：
```php
$user = UserModel::select(["uid", "nickname"])->find(1);

// select `uid`, `nickname`, `is_vip` from `user` where `user`.`uid` = '1' limit 1
```

## 场景二
一对多关联（用户表与文章表）：如何获最新一条记录或统计关联数据的合计。

以用户与文章之间的一对多关联为例，如果用户列表需要返回用户发布最新文章的标题，那么该如何进行查询？

这个场景下使用连接查询是不行的，因为涉及到被驱动表的排序和限定查询问题。

```php
public function index()
{
    $users = User::addSelect([
        'last_post_title' => Post::select(['title'])
            ->whereColumn('uid', 'users.uid')
            ->orderByDesc('created_at')
            ->limit(1)
    ])
      ->orderByDesc('uid')
      ->get();
}
```

生成 SQL 如下：
```sql
select (select title from `posts` where `uid` = `user`.`uid` order by `created_at` desc limit 1 ) as `last_post_title`,`user`.*  from `user` order by `uid` desc ;
```

对某个字段进行合计也是一样的：
```php
public function index()
{
UserModel::addSelect([
      "amount" => OrderModel::selectRaw('sum(total_price)')
            ->whereColumn('uid', 'user.uid')
            ->orderByDesc("total_price")
            ->limit(1)
    ])
      ->orderByDesc("amount")
      ->get();
}
```

生成 SQL 如下：
```sql
select (select SUM(total_price) from `order` where `uid` = `user`.`uid` order by `total_price` desc limit 1 ) as `amount`,`user`.* from `user` order by `amount` desc ;
```

## 场景三
聚合统计：统计订单表中不同状态下的订单数量。

思路一：对订单状态进行分组，然后通过代码逻辑统计不同状态下订单数量。
```php
public function index()
{
    AuctionOrderModel::selectRaw("count(order_id)")
          ->groupBy("auction_order_status")
          ->get();
}
```

生成 SQL：
```mysql
select count(order_id) from `auction_order` where `auction_order`.`is_deleted` = '0' group by `auction_order_status
```

思路一：将多次聚合统计查询合并为一次查询：
```php
public function index()
{
    OrderModel::selectRaw('COUNT(CASE WHEN `user_order_status` = 0 then 1 END) AS draft_count')
            ->selectRaw('COUNT(CASE WHEN `user_order_status` = 1 then 1 END) AS audit_count')
            ->selectRaw('COUNT(CASE WHEN `user_order_status` = 2 then 1 END) AS normal_count')
            ->first();
}
```

生成 SQL 如下：
```sql
select COUNT(CASE WHEN `user_order_status` = 0 then 1 END) AS draft_count, COUNT(CASE WHEN `user_order_status` = 1 then 1 END) AS audit_count, COUNT(CASE WHEN `user_order_status` = 2 then 1 END) AS normal_count from `order` limit 1
```

## 场景四
一对一关联排序（用户主表与用户辅表）：用户列表可根据用户辅表的某个字段进行排序。

思路一：子查询
```php
public function index()
{
UserModel::select(['uid', 'nickname'])
      ->orderBy(UserInfoModel::select('broken_number')
          ->whereColumn('uid', 'user.uid')
          ->orderBy('broken_number')
          ->limit(1)
      )
      ->paginate(20);
}
```

生成 SQL 如下：
```sql
select `uid`, `nickname` from `user` order by (select `broken_number` from `user_info` where `uid` = `user`.`uid` order by `broken_number` desc limit 1) desc limit 20 offset 0;
```

思路二：关联查询
```php
public function index()
{
    UserModel::select("user.*")
        ->join('user_info', 'user_info.uid', '=', 'user.uid')
        ->orderBy('user_info.broken_number')
        ->paginate(20);
}
```

生成 SQL 如下：
```sql
 select `user`.* from `user` inner join `user_info` on `user_info`.`uid` = `user`.`uid` order by `user_info`.`broken_number` asc limit 20 offset 0
```

## 场景五
一对多关联排序（用户表与订单表）：用户列表展示用户对应创建订单的金额，并根据金额大小进行排序。

思路一：子查询：
```php
public function index()
{
    UserModel::addSelect([
       "cost_amount" => OrderModel::selectRaw('sum(total_price)')
            ->whereColumn('uid', 'user.uid')
            ->orderByDesc("total_price")
            ->limit(1)
    ])
        ->orderByDesc("cost_amount")
        ->paginate(20);
}
```

生成 SQL 如下：
```php
select `user`.*, (select sum(total_price) from `order` where `uid` = `user`.`uid` and `order`.`is_deleted` = '0' order by `total_price` desc limit 1) as `cost_amount` from `user` order by `cost_amount` desc limit 20 offset 
```
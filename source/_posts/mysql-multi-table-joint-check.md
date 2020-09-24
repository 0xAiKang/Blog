---
title: Mysql 多表联查
date: 2020-09-24 18:56:22
tags: ["Mysql"]
categories: ["Mysql"]
---

Mysql 的两张表联表查询可能大家都知道怎么查，但如果是三张表或者是更多张表呢？

<!-- more -->

其实不管是两张表还是三张表还是N 张表都是一样的。

### 多表联查

```
# 语法一：
select t1.*, t2.*, t3.* 
from table1 t1, table2 t2, table3 t3
where t1.id = t2.id and t1.id = t3.id;

# 语法二：
select t1.*, t2.*, t3.* 
from table t1 inner join table2 t2 
on t1.id = t2.id 
inner join table3 t3 
on t1.id = t3.id;
```

有几点需要注意：
1. 上面的id 并不一定非要使用id，可以是任何有关联性的其他字段
2. 如果表名是关键字，那么需要查询时在这个关键字上加反引号，如：\`order\`
3. `inner join` 可以根据实际情况可以换成`left join`、`right join` 
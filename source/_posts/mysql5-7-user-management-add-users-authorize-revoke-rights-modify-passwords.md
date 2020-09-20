---
title: mysql5.7用户管理：添加用户、授权、撤权、修改密码
date: 2020-09-20 23:42:13
tags: ["Mysql"]
categories: ["Mysql"]
---

因为Mysql 5.7 是目前使用最多的数据库，而5.7 在某些地方又和其他版本有所不同，所以记录一下。

<!-- more -->

## 创建用户

```
# 语法：CREATE USER 'username'@'host' IDENTIFIED BY 'password';

mysql>  CREATE USER 'boo'@'localhost' IDENTIFIED BY '122410';
```
host 参数说明：
* `%`：匹配所有主机
* `localhost`：当前主机，localhost 不会被解析成IP地址，而是通过UNIXsocket 连接
* `127.0.0.1`：当前主机，通过TCP/IP 协议连接
* `::1`：当前主机，兼容支持ipv6

此时还没有授权，只能登陆，无法做其余操作

## 用户授权

```
# 创建完成之后授权
mysql> grant all privileges ON `dbName`.* TO 'username'@'host';

# 创建用户同时授权
mysql> grant all privileges on dbName.* to 'username'@'host' identified by 'password';

# 刷新权限
mysql>  flush privileges;

# 查看用户所有权限
mysql> show grants for dev@'%';

# 撤消用户授权，撤消要求各参数与授权时使用的一致，可以先查看授权再撤消
mysql> revoke privileges ON dbName.* FROM 'username'@'host';
```
privileges 参数说明：
* `all privileges`: 所有权限；
* `select`: 查询；
* `insert`: 新增记录;
* `update`: 更新记录；
* `delete`: 删除记录；
* `create`: 创建表；
* `drop`: 删除表；
* `alter`: 修改表结构；
* `index`: 索引相关权限；
* `execute`: 执行存储过程与call函数
* `references`： 外键相关；
* `create temporary tables`：创建临时表；
* `lock tables`：锁表；
* `create view`：创建视图；
* `show view`：查看视图结构；
* `trigger`: 触发器；

dbName 可以是某个库（`database`），也可以是具体到某张表（`database.table`），也可以是所整个数据库（`*`）。

## 修改密码
```
# 修改自己的密码
mysql> set password=password('newpassword');

# 修改别人密码——方法1
mysql> set password for 'username'@'host' = password('newpassword');

# 修改别人密码——方法2: 适用mysql5.7以前的版本，5.7以后的版本中mysql.user表没有了password字段
mysql> update mysq.user set password=password('newpassword') where user='user' and host='host';

# 修改别人密码——方法3：适用mysql5.7
mysql> update mysql.user set authentication_string=password('newpassword') where user='root';

# 修改别人密码——方法4
mysql> alter user 'test'@'%' identified by 'newpassword';
```

## 删除用户

```
mysql> DROP USER 'username'@'host';
```

不建议直接通过修改`mysql.user`表去操作用户。

### 参考链接
* [mysql5.7用户管理：添加用户、授权、撤权、修改密码](https://blog.csdn.net/yu12377/article/details/78214336)
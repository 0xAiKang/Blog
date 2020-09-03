---
title: Mysql 主从架构配置
date: 2020-09-03 23:40:16
tags: ["Mysql"]
categories: ["Mysql"]
---

Mysql 主从配置是数据库同步的必要步骤。

<!-- more -->

主机环境：
* Ubuntu 18.04 LTS
* Mysql 5.7

下面会将主数据库简称为Master，从数据库简称为 Slave。

## 配置Master
```
# vim /etc/mysql/mysql.conf.d/mysqld.cnf

# 打开二进制日志
[mysqld]
server_id=1
log-bin=master-bin
log-bin-index=master-bin.index
```

创建同步用户，并赋予权限（如果从服务器以reql 这个账号进行连接，就赋予同步数据库的权限，并且这个权限是所有数据库的所有数据表）
```
$ mysql -uroot -p 
mysql> create user repl;
mysql> grant replication slave on *.* to 'user'@'your_slave_addr' identified by 'password'
mysql> flush privileges;
```
上面的IP 是指 Slave 服务器的IP 地址。
重启Mysql 服务。

查看Master 配置：
```
mysql> show master status;
```

## Salve 配置
```
# vim /etc/mysql/mysql.conf.d/mysqld.cnf

# 打开relay 日志
[mysqld]
server_id=2
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin
```
重启Mysql 服务。

指定Master 主机
```
$ mysql -uroot -p
mysql> change master to master_host="your master ip ", master_port=3306, master_user='repl',master_password='password',master_log_file='master-bin.000001',master_log_pos=0;
```
参数说明：
* `master_host`：Master∑主机的外网IP 地址
* `master_port`：端口
* `master_user`：Master主机上进行同步的用户
* `master_password`：密码
* `master_log_file`：Master 输出的二进制文件的名称（在Master 主机上使用`show master status`命令查看）
* `master_log_pos`：哪里开始同步

开启主从同步
```
mysql> start slave;
```

查看从库同步状态
```
mysql> show slave status;
```

### 可能会遇到的问题

> Last_Errno: 1146
  Last_Error: Error executing row event: 'Table 'panda.t' doesn't exist'
  
解决办法：使用`slave-skip-errors` 参数跳过该错误。

```
# vim /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
slave_skip_errors=1146
```
重启从库即可。
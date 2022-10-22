---
title: ElasticSearch 快速上手
date: 2022-10-22 17:58:44
tags: ["ElasticSearch"]
categories: ["ElasticSearch"]
---

[Elasticsearch](https://www.elastic.co/cn/) 是目前全文搜索引擎的首选，它可以快速地储存、搜索和分析海量数据。

ES 底层是开源库 Lucene。但是没法直接用 Lucene，必须自己写代码去调用它的接口。Elastic 是 Lucene 的封装，提供了 REST API 的操作接口，开箱即用。

<!-- more -->

## 安装
ElasticSearch 需要 Java 8 环境。如果你的机器还没安装 Java，可以进行[下载安装](https://www.java.com/en/download/)。

ES 的安装比较简单，直接[下载](https://www.elastic.co/cn/downloads/elasticsearch)对应版本的压缩包解压即可：
```bash
curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.3-darwin-x86_64.tar.gz
curl https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.3-darwin-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
tar -xzf elasticsearch-8.4.3-darwin-x86_64.tar.gz
```
这里下载安装的是最新版本 `8.4.3`。

首次解压完并不能直接运行，需要稍微修改一些配置（如果你的机器有配置证书则可以忽略，这是因为 ES 默认开启了 `ssl` 认证）：
```
// vim elasticsearch.yml
xpack.security.enabled: false
```

进入解压后的目录，运行下面的命令，启动 ES：
```bash
bin/elasticsearch
```

如果这时报错 "max virtual memory areas vm.maxmapcount [65530] is too low"，要运行下面的命令。
```bash
sudo sysctl -w vm.max_map_count=262144
```

如果一切正常，Elastic 就会在默认的 9200 端口运行，访问`localhost:9200`会返回如下信息：
```json
{
  "name" : "192.168.123.11",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "GUgWHb9ERuOo230FP2os4g",
  "version" : {
    "number" : "8.4.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "42f05b9372a9a4a470db3b52817899b99a76ee73",
    "build_date" : "2022-10-04T07:17:24.662462378Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

```

## head 插件安装
[elasticsearch-head](https://github.com/mobz/elasticsearch-head) 是一个ES 集群的 Web 前端控制台，可以可视化管理 ES。

安装也是非常简单，直接下载解压运行即可：
```
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
```

elasticsearch-head 默认监听 9100 端口，正常访问 `localhost:9100` 会看到如下界面：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221021211444.png)
注意看，我这里的集群健康值是 red，索引的旁边出现了一个 `unassigned`，出现 `unassigned` 的原因通常有：
1. 磁盘空间不足（控制磁盘使用的高水位。默认为90%）
2. nodes 数小于分片副本数
3. 节点失联

健康值有三个值：
* green
* yellow
* red

如果处于 red，很多操作是做不了的，我这里是因为磁盘空间不足而导致的，释放掉磁盘空间之后便恢复 green 了：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221022164249.png)
## 基本概念

### Node 与 Cluster
ES 本质上是一个分布式的数据库，允许多台服务器协同工作，每台服务器也可以同时运行多个实例。

单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。

### Index
Index 是 ES 的核心概念，ES 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

ES 数据管理的顶层单位就叫做 Index（索引）。它是单个数据库的同义词。每个 Index （即数据库）的名字必须是小写。

### Document
Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

Document 使用 JSON 格式表示，下面是一个例子：
```json
{
  "name": "李四",
  "gender": "男"
}
```

### Type

## 数据操作

### 创建索引

创建 Index，可以直接向 ES 服务器发出 PUT 请求。
```bash
# 创建一个 weather 的 Index
curl -X PUT 'localhost:9200/customer'
```

### 删除索引
通过 DELETE 请求删除 Index
```bash
curl -X DELETE 'localhost:9200/weather'
```

### 新增记录
 
```php
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "李四",
  "gender": "男"
}'
```
`pretty` 参数的作用是以易读的格式返回。

注意，这里请求地址是`customer/_doc/1`，最后的1是该条记录的 Id。它不一定是数字，任意字符串（比如abc）都可以。

新增记录的时候，也可以不指定 Id，这时要改成 POST 请求。

```bash
curl -X POST "localhost:9200/customer/_doc?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "张三",
  "gender": "男"
}'

{
  "_index" : "customer",
  "_id" : "-c3r_oMBDokqbW4aIGss",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```
需要注意的是，使用该请求方式时，如果对应的索引不存在（例子中是`customer`），ES 则会自动创建该索引。

### 查看记录

通过 `elasticsearch-head` 查看数据：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20221022171045.png)

通过终端指定 Document ID 查看对应的记录：
```bash
curl -X GET "localhost:9200/customer/_doc/1?pretty"
{
  "_index" : "customer",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "李四",
    "gender" : "男"
  }
}
```

返回的数据中，found字段表示查询成功，`_source` 字段返回原始记录。如果 Id 不正确，就查不到数据，found字段就是false。

### 更新记录
```bash
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "王五",
  "gender": "男"
}'
{
  "_index" : "customer",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

上面代码中，将原始数据从"李四"改成“王五"。 返回结果里面，有几个字段发生了变化。

```
"_version" : 2,
"result" : "updated",
"created" : false
```

可以看到，记录的 Id 没变，但是版本（version）从1变成2，操作类型（result）从created变成updated，created字段变成false，因为这次不是新建记录。

### 删除记录
删除记录就是发出 DELETE 请求。
```bash
curl -X DELETE 'localhost:9200/customer/_doc/1'
```

## 数据查询

### 返回所有记录
最新版本的 ES，通过请求`/Index/_search`，就会返回对应索引下的所有记录。
```php
curl -X GET "localhost:9200/customer/_search?pretty"
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_id" : "-c3r_oMBDokqbW4aIGss",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三",
          "gender" : "男"
        }
      },
      {
        "_index" : "customer",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "王五",
          "gender" : "女"
        }
      }
    ]
  }
}
```

上面代码中，返回结果：
* took 字段表示该操作的耗时（单位为毫秒）
* timed_out 字段表示是否超时
* hits 字段表示命中的记录

hits 的子字段的含义如下：
* total：返回记录数
* max_score：最高的匹配程度
* hits：返回的记录组成的数组

返回的记录中，每条记录都有一个 `_score` 字段，表示匹配的程序，默认是按照这个字段降序排列。

### 全文搜索
Elastic 的查询非常特别，使用自己的[查询语法](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/query-dsl.html)，要求 GET 请求带有数据体。

```bash
curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'{
    "query":{
        "bool":{
            "must":[
                {
                    "term":{
                        "name.keyword":"张三"
                    }
                }
            ],
            "must_not":[

            ],
            "should":[

            ]
        }
    },
    "from":0,
    "size":10,
    "sort":[

    ],
    "aggs":{

    }
}'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.6931471,
    "hits" : [
      {
        "_index" : "customer",
        "_id" : "-c3r_oMBDokqbW4aIGss",
        "_score" : 0.6931471,
        "_source" : {
          "name" : "张三",
          "gender" : "男"
        }
      }
    ]
  }
}
```

## 参考链接
* [ElasticSearch Getting started guides](https://www.elastic.co/guide/en/welcome-to-elastic/current/getting-started-guides.html#getting-started-guides)
* [ElasticSearch 查询语法](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/query-dsl.html)
* [全文搜索引擎 Elasticsearch 入门教程](https://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
* [ES 的 unassigned shards 核心处理方案](https://blog.csdn.net/qq_33999844/article/details/108907902)
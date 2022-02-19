---
title: ElasticSearch (III) -- Elastic 使用入门 + Postman 使用
date: 2022-01-31 16:36:27
tags: 
- ElasticSearch
- Apache
categories:
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch/elastic_search_top.png
---

### ElasticSearch 使用入门

### RESTful api 的创建与使用

From Wiki: 

> REpresentational State Transfer, REST 是一种软件架构风格，定义了一组创建 Web 服务的约束。RESTful Web 服务允许请求系统通过使用统一和预定义的无状态操作集来访问和操作 Web 资源的文本表示。

所以，REST 到底是什么？就是客户端向服务端请求访问制定数据，或者在服务端推送、保存数据，服务端相应客户端请求的过程。从编程的角度可能更加容易解释，服务端提供一个端点 URL 给客户端，客户端连接这个端点并发送数据（REST 不负责存储携带的数据）、服务端返回相应。这样的流程就是 REST 的流程。这里面主要有几个模块需要简单了解。

1. 资源 Resource：真是的对象数据，可以是集合也可以是单个个体，每一种资源都有特定的 URI（统一资源定位符）与之对应。资源里面还可以包含子资源。

2. 表现形式 Representational：信息实体，是资源具体呈现的形式，比如 json，xml 等。

3. 状态转移 State Transfer：通过HTTP 动词实现增删查改等操作，从而引起资源状态的改变。这个处于 Server 端。

#### REST 接口规范

REST 接口指令主要由四部分组成：动作 + 路径 + 过滤信息 + 状态码，动作主要是用于 specify 增删查改等操作，而路径又称 "endpoint"，是 API 的具体网址。

##### 动作

- GET: 请求服务器获取特定资源

- POST：在服务器上创建一个新的资源

- PUT：更新服务器上的资源（倾向于整体更新）

- PATCH：更新服务器上的资源 （倾向于部分更新）

- DELETE：从服务器删除特定的资源

##### 路径

对路径在开发过程中会有一定的规范要求

1. 网址中不能有动词，只能有名词，API 中的名词也应该使用复数。如果 API 调用并不涉及资源（如计算，翻译等操作），可以使用动词。

2. 不用大写字母，建议用 `-` 而非 `_`

##### 过滤信息 (Filtering)

在查询是可以添加特定的条件，建议使用 url 参数的形式。比如

```shell
GET /merchandise?is_live=true&size=10
```

##### 状态码

状态码范围：

| 2xx：成功 | 3xx：重定向   | 4xx：客户端错误  | 5xx：服务器错误 |
| ------ | --------- | ---------- | --------- |
| 200 成功 | 301 永久重定向 | 400 错误请求   | 500 服务器错误 |
| 201 创建 | 304 资源未修改 | 401 未授权    | 502 网关错误  |
|        |           | 403 禁止访问   | 504 网关超时  |
|        |           | 404 未找到    |           |
|        |           | 405 请求方法不对 |           |

### Elasticsearch  接口语法

```shell
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

其中：

| 参数           | 解释                                                     |
| ------------ | ------------------------------------------------------ |
| VERB         | 适当的 HTTP 方法 或 谓词 : GET 、 POST 、 PUT 、 HEAD 或者 DELETE 。 |
| PROTOCOL     | http 或者 https （如果在 Elasticsearch 前面有一个 https 代理）       |
| HOST         | Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。    |
| PORT         | 运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。                |
| PATH         | API 的终端路径（例如 _count 将返回集群中文档数量）。                       |
| QUERY_STRING | 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)     |
| BODY         | 一个 JSON 格式的请求体 (如果请求需要的话)                              |

比如：

```shell
curl -XGET -HContent-Type:application/json 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

它会返回整个 json 格式的结果

```json
{
  "count" : 0,
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "skipped" : 0,
    "failed" : 0
  }
}
```

#### POSTMAN 使用

Postman 是一个非常好用的对容器服务做增删查改操作的接口测试工具。其下载地址为 [官方网址](https://www.postman.com/downloads/)。下载后直接解压即可使用，注意将解压后的执行文件移至「应用」文件夹。

##### 界面基础功能介绍

刚进入界面时，需要自己新建一个 Collection 与 Workspace，然后会进入如下界面。

![postman_intro_1.png](https://jason24-zeng.github.io/img/ElasticSearch-III/postman_intro_1.png)

主要有四个部分：

1. 左上 Collection（可以理解成文件夹，可以把一个项目的请求放到 Collections 中方便管理）， History 等 Toolbox，用于查找相关的项目，接口集或者历史等

2. 右上请求方式与请求网址等。用于 specify 测试的接口。

3. 右中请求参数，可以使 key-value 或者其他任意形式的参数，可以理解成过滤信息。

4. 右下相应内容，一般使用 Pretty 格式化响应内容。里面包含 HTTP 响应测试码，响应时间与大小等。

##### 集群设置-自动创建索引

集群在初始化设置时，不会有 persistent 这个参数，导致使用默认参数 `persistent = true`，这时，当我们指定一个不存在的索引，新增文档就会报错。而当 `persistent = false` 时，则会新建文档，同时创建这个不存在的索引。

1. 查看集群设置
   
   ```shell
   GET http://localhost:9200/_cluster/settings
   ```
   
   response 为
   
   ```json
   {
       "persistent": {},
        "transient": {}
   }
   ```

2. 修改集群设置
   操作如下图，可看到返回体中 `persistent.action.auto_create_index=false`
   ![postman_intro_2.png](https://jason24-zeng.github.io/img/ElasticSearch-III/postman_intro_2.png)

##### 索引操作 - index

###### 创建索引

动作 + 路径

```shell
PUT http://localhost:9200/nba
```

JSON 格式的请求体

```json
{
    "mappings": {
        "properties": {
            "name": {
                "type": "text"
            },
            "team_name": {
                "type": "text"
            },
            "position": {
                "type": "text"
            },
            "play_year": {
                "type": "long"
            },
            "jerse_no": {
                "type": "keyword"
            }
        }
    }
}
```

图示

![postman_intro_3.png](https://jason24-zeng.github.io/img/ElasticSearch-III/postman_intro_3.png)

###### 获取索引

```shell
GET http://localhost:9200/nba
```

###### 关闭索引

```shell
POST http://localhost:9200/nba/_close
```

###### 删除索引

```shell
DELETE http://localhost:9200/nba/
```

##### 映射操作

###### 创建映射

```shell
PUT http://localhost:9200/nba/_mapping
```

```json
{
    "properties": {
        "name": {
            "type": "text"
        },
        "team_name": {
            "type": "text"
        },
        "position": {
            "type": "keyword"
        },
        "play_year": {
            "type": "keyword"
        },
        "jerse_no": {
            "type": "keyword"
        },
        "country": {
            "type" : "keyword"
        }
    }
}
```

需要注意到如果不是新建索引，而是在之前索引的基础上进行修改，会有如下报错。

```shell
mapper [play_year] cannot be changed from type [long] to [keyword]
```

表明我们对已经构建的索引类型进行修改。

###### 获取索引

```shell
GET http://localhost:9200/nba/_mapping
```

##### 文档操作

###### 新增文档-指定 ID

```shell
PUT http://localhost:9200/nba/_doc/23
```

```json
{
    "name" : "Lebron 詹姆斯",
    "team_name" : "Los Angle Lakers",
    "position" : "small forward",
    "play_year" : 15,
    "jerse_no" : "23"
}
```

##### 新增文档-不指定 ID

注意与上一节的区别是，用的操作类型由 `PUT` 改成了 `POST`

```shell
POST http://localhost:9200/nba/_doc
```

```json
{
    "name" : "James 哈登",
    "team_name" : "Houston Rockets",
    "position" : "shooting guard",
    "play_year" : 10,
    "jerse_no" : "13",
    "country" : "庞各庄"
}
```

响应体信息

```json
{
    "_index": "nba",
    "_type": "_doc",
    "_id": "htJWtX4BnmkKdz--Ip7s",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

其他的相关内容可以查看 [ElasticSearch7.2简单命令实操(postman版)](https://www.cnblogs.com/geoffreygao/p/13889696.html)。例子非常生动形象。

### Reference

[REST API 教程：REST 客户端，REST 服务及 API 调用（含代码示例）](https://chinese.freecodecamp.org/news/rest-api-tutorial-rest-client-rest-service-and-api-calls-explained-with-code-examples/)

[RestFul API 简明教程](https://www.woshinlper.com/system-design/restful-api/)

[ElasticSearch7.2简单命令实操(postman版)](https://www.cnblogs.com/geoffreygao/p/13889696.html)

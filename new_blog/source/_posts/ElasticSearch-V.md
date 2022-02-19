---
title: ElasticSearch (V) -- 中文分词
date: 2022-02-02 18:28:06
tags:
- ElasticSearch
- Apache
- Ik 分词器
categories:
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch/elastic_search_top.png
---

这一章节主要讲解如何下载安装与使用分词器。

ElasticSearch 镜像里有各式各样的英文分词器，比如特定语言分词器，正则分词器，stop 分词器，空白分词器，简单分词器，标准分词器等。但是对于中文分词的支持不是很好，我们需要单独下载安装中文分词的插件，这里介绍对中文分词器 IK-analyzer 的下载安装。

### 分词器下载安装

#### 使用 docker-compose 对集群进行安装

需要单独给每个节点安装，使用指令

```shell
docker-compose exec es01 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.16.2/elasticsearch-analysis-ik-7.16.2.zip
docker-compose exec es02 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.16.2/elasticsearch-analysis-ik-7.16.2.zip
```

然后需要重启 es 容器

```shell
docker-compose restart es01
docker-compose restart es02
```

需要注意，分词器的版本需要与 es 的镜像版本保持一致，比如上面，我们使用的 elasticsearch 的版本就是 7.16.2。镜像更新后，其他 plugin 也需要相应得进行更新。否则可能有如下报错：

```textfile
Exception in thread "main" java.lang.IllegalArgumentException: Plugin [analysis-ik] was built for Elasticsearch version 7.1.1 but version 7.16.2 is running
```

#### 进入容器内对单点进行安装

另一种方法稍微麻烦一点，不使用 `docker-compose` 命令，整体操作如下

```shell
// 0x01. 进入容器
docker exec -it elasticsearch_01 /bin/bash
// 0x02. 使用 bin 目录的 elasticsearch-plugin install 安装插件
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.1/elasticsearch-analysis-ik-7.16.2.zip
// 0x03. cmd + D 退出并重启容器
docker restart elasticsearch_01
```

#### 验证分词插件是否成功

首先先通过重启后的日志检查是否 load 了 `analysis-ik` 这个插件，具体如下图：

![ik_installed_info.png](/Users/zijianzeng/Documents/hexo/new_blog/source/img/ElasticSearch-V/ik_installed_info.png)

则表示 load 成功了，再验证能否正常使用该插件。

使用 postman 发送分析请求

```shell
GET http://127.0.0.1:9200/_analyze
```

使用 `ik_smart` 分词，请求体为

```json
{
    "analyzer": "ik_smart",
    "text": "我们是中国软件工程师"
}
```

或使用 `ik_max_word` 分词

```json
{
    "analyzer": "ik_max_word",
    "text": "我们是中国软件工程师"
}
```

最终 `ik_smart` 返回结果是

```json
{
    "tokens": [
        {
            "token": "我们",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "是",
            "start_offset": 2,
            "end_offset": 3,
            "type": "CN_CHAR",
            "position": 1
        },
        {
            "token": "中国",
            "start_offset": 3,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "软件",
            "start_offset": 5,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "工程师",
            "start_offset": 7,
            "end_offset": 10,
            "type": "CN_WORD",
            "position": 4
        }
    ]
}
```

`ik_max_word` 与之稍有区别，如下：

```json
{
    "tokens": [
        {
            "token": "我们",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "是",
            "start_offset": 2,
            "end_offset": 3,
            "type": "CN_CHAR",
            "position": 1
        },
        {
            "token": "中国",
            "start_offset": 3,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "软件工程",
            "start_offset": 5,
            "end_offset": 9,
            "type": "CN_WORD",
            "position": 3
        },
        {
            "token": "软件",
            "start_offset": 5,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 4
        },
        {
            "token": "工程师",
            "start_offset": 7,
            "end_offset": 10,
            "type": "CN_WORD",
            "position": 5
        },
        {
            "token": "工程",
            "start_offset": 7,
            "end_offset": 9,
            "type": "CN_WORD",
            "position": 6
        },
        {
            "token": "师",
            "start_offset": 9,
            "end_offset": 10,
            "type": "CN_CHAR",
            "position": 7
        }
    ]
}
```

而非报错信息，即证明分词器可用。

### 创建索引时设置分词器

操作例子如下

```shell
PUT http://elasticsearch-1:9200/analyzer_index
```

请求体为

```json
{
    "settings": {
        "analysis": {
            "analyzer": {
               "my_analyzer": {
                    "type": "whitespace"
                }
            }
        }
    },
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
            },
            "title": {
                "type": "text",
                "analyzer": "my_analyzer"
            }
        }
    }
}
```

注意 mapping 中的 title 使用了 analyzer 配置。当然，在 `"title"`里面设置 analyzer 还可以直接设置已有的 plugin，比如 `"ik_max_word"`。

### Reference

[ElasticSearch7.2简单命令实操(postman版)](https://www.cnblogs.com/geoffreygao/p/13889696.html)

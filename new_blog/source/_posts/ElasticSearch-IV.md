---
title: ElasticSearch (IV) -- Elastic 搜索查询语法
date: 2022-02-02 12:27:05
tags:
- ElasticSearch
- Apache
categories:
- Dev Tools
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch/elastic_search_top.png
---

### DSL 搜索

DSL 搜索时 Domain Speicific Search。我们不必如下式全局搜索展示的一样在搜索时写一连串的条件体。这样的做法在于方便 debug。

```shell
GET http://localhost:9200/nba/_search?q=play_year:10&q=name:库里
```

#### DSL 搜索形式

DSL 以 JSON 请求体的形式发起搜索请求，允许更加复杂，强大的查询

```shell
POST http://localhost:9200/nba/_doc/_search
#  请求
{
    "query": {
        "match" : { # match 只是查询中的一种
            "play_year" : 10
        }
    }
}
```

请求与返回结果如下：

![DSL_1.png](https://jason24-zeng.github.io/img/ElasticSearch-IV/DSL_1.png)

#### 多条件判断

复杂一点的范围查询，比如找出 `play_year > 5` 且 `position = shooting guard` 的 NBA 球员

```shell
POST http://localhost:9200/nba/_doc/_search
```

请求体：

```json
{
    "query": {
        "bool" : {
            "filter" : {
                "range" : {
                    "play_year" : {
                        "gt" : 5
                    }
                }
            },
            "must" : {
                "match" : {
                    "position" : "shooting guard"
                }
            }
        }
    }
}
```

这样搜索就可以了。返回结果

![DSL_2.png](https://jason24-zeng.github.io/img/ElasticSearch-IV/DSL_2.png)

#### 或逻辑判断

如果我们想返回 `name=Lebron 詹姆斯` 和 `name=James 哈登`两个匹配的结果，可以输入

```shell
POST http://localhost:9200/nba/_doc/_search
```

```json
{
    "query": {
        "match" : {
            "name" : "Lebron James"
        }
    }
}
```

返回结果如下，因为 match 是指找到所有能匹配请求体中 string 的结果。

![DSL_3.png](https://jason24-zeng.github.io/img/ElasticSearch-IV/DSL_3.png)

### 高亮显式

想要高亮否一些 fields，可以通过在请求体中加入一个与 `query` 同级的 `highlight` 请求体

```shell
POST http://localhost:9200/nba/_doc/_search
```

```json
{
    "query": {
        "match" : { 
            "position" : "shooting guard"
        }
    },
    "highlight" : {
        "fields" : {
            "position" : {}
        }
    }
}
```

### 聚合

类似于 SQL 中的 group by 操作

```shell
POST http://localhost:9200/nba/_doc/_search
```

```json
{
    "aggs" : {
        "all_interests" : {
            "terms" : {
                "field" : "play_year"
            }
        }
    }
}
```

### 批量查询或插入数据

#### 批量查询数据

```shell
POST http://localhost:9200/nba/_doc/_mget
```

```json
{
    "ids": [
        "2",
        "23"
    ]
}
```

其中有数据不存在，不影响结果的返回，只是对该 id 的返回体 `found` 为 `false`

#### 批量插入、修改、删除数据

使用 `_bulk` api 完成如下请求格式

```json
{action : {metadata} } \n
{request body        } \n
{action : {metadata} } \n
{request body        } \n
...
```

###### 插入请求体示例

```json
{"create":{"_index":"nba","_type":"_doc","_id":3}}
{"name": "Anthony Davids", "team_name":"Los Angles Lakers", "position":"Center", "play_year":9}
{"create":{"_index":"nba","_type":"_doc","_id": 12}}
{"name": "Russel Westbrook", "team_name":"Los Angles Lakers", "position":"Point Guard", "play_year":10}
```

注意每一行都需要 `\n`

###### 删除请求体示例

```json
{"delete":{"_index":"nba","_type":"_doc","_id":2}}
{"delete":{"_index":"nba","_type":"_doc","_id":"htJWtX4BnmkKdz--Ip7s"}}
```

批量请求大小多少才能使性能最高？并不是越大越好，因为请求越大，给其他请求可用的内存就越小，可以找到一个请求 bulk，超过这个，性能就不再提升。而这个 bulk 数与我们的硬件，文档大小和复杂度以及索引和搜索的负载有关。一个好的批次最好保持在 5 - 15 MB 之间。

### 分页

和 SQL 中的 `LIMIT` 关键字相似，返回只有一页的结果， Elasticsearch 接受 `from` 和 `size` 参数，这两个参数和 C++ 中的 substr 一致，`from` 表示起始位置与 `0` 之间的 offset，而 `size` 则表示需要获取的结果数。

如果我们想要每页显示 5 个结果，页码从 1 到 3，请求结果为 

```shell
GET .../_search?size=5
GET .../_search?size=5&from=5
GET .../_search?size=5&from=10
```

**Warning!** ：需要注意，在集群中做深度分页会有问题。假设我们请求第 1000 页的 10 条结果，每页的 size 为 50 个，那意味着我们会请求节点排序 50050 条结果，但最终丢弃其中的 50040 条结果，排序结果的花销随着分页的深入而成倍增长。因此网络搜索引擎中任何语句都不能返回多余 1000 个结果。

### 映射

Elasticsearch 可以自动判断类型，但有时其判断的类型和实际需求不符，这时我们需要明确字段类型。

自动判断的规则为：

| JSON 类型                           | Field 类型    |
| --------------------------------- | ----------- |
| Boolean: `true` of `false`        | `"boolean"` |
| Whole number: `123`               | `"long"`    |
| Floating point: `123.45`          | `"double"`  |
| String.valid date: `"2014-09-15"` | `"date"`    |
| String: `"foo bar"`               | `"string"`  |

Elasticsearch 中支持的类型：

| 类型             | 表示的数据类型                          |
| -------------- | -------------------------------- |
| String         | `"string","text","keyword"`      |
| Whole number   | `"long","byte","integer","long"` |
| Floating point | `"double","double"`              |
| Date           | `"date"`                         |
| Boolean        | `"boolean"`                      |

Notice:

1. Elasticsearch 5.x 不再支持`string`，而用 `text` 与 `keyword` 类型代替。

2. 当一个字段要被全文搜索时 (需要分词)，比如 Email 内容，产品描述，应该使用 `text` 类型。这样字段内容会被分析，在生成倒排索引之前，字符串会被分析器分成一个个词，`text` 类型的字段不用于排序，很少用于聚合。

3. `keyword` 类型则适用于索引结构化的字段，比如 Email 地址，主机名，状态码和标签。如果字段需要进行过滤、排序、聚合。`keyword` 类型的字段只能通过精确值搜到。

#### 创建明确类型的索引

操作 

```shell
PUT http://localhost:9200/itcast
```

```json
{
    "settings" : {
        "index" : {
            "number_of_shards" : "2",
            "number_of_replicas" : "0"
        }
    },
    "mappings" : {
        "person" : {
            "properties" : {
                "name" : {
                    "type" : "text"
                },
                "age" : {
                    "type" : "integer"
                },
                "mail" : {
                    "type" : "keyword"
                },
                "hobby" : {
                    "type" : "text"
                }
            }
        }
    } 
}
```

上面这个例子在 ES 7.x 中是会报错的，报错为 

```textile
{
    "error": {
        "root_cause": [
            {
                "type": "mapper_parsing_exception",
                "reason": "Root mapping definition has unsupported parameters:  [person : {properties={mail={type=keyword}, name={type=text}, age={type=integer}, hobby={type=text}}}]"
            }
        ],
        "type": "mapper_parsing_exception",
        "reason": "Failed to parse mapping [_doc]: Root mapping definition has unsupported parameters:  [person : {properties={mail={type=keyword}, name={type=text}, age={type=integer}, hobby={type=text}}}]",
        "caused_by": {
            "type": "mapper_parsing_exception",
            "reason": "Root mapping definition has unsupported parameters:  [person : {properties={mail={type=keyword}, name={type=text}, age={type=integer}, hobby={type=text}}}]"
        }
    },
    "status": 400
}
```

报错原因是， ES 7 之后，对于 `document_type` 这个 mapping type 被移除了，具体的细节可以查询 [Removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)

所以正确的构造请求体为

```json
{
    "settings": {
        "index": {
            "number_of_shards": "2",
            "number_of_replicas": "0"
        }
    },
    "mappings": {
        "properties": {
            "name": {
                "type": "text"
            },
            "age": {
                "type": "integer"
            },
            "mail": {
                "type": "keyword"
            },
            "hobby": {
                "type": "text"
            }
        }
    }
}
```

这个明确类型最大的效果实际上是在分词那部分。我们需要给我们需要的字段设定 `text` 类型，因为 `keyword`类型无法分词。

### 结构化查询

查询操作使用相同，只是请求体内容不同

```shell
PUT http://localhost:9200/nba/_doc/_search
```

#### term 查询

`term` 主要是用于精确匹配一些数字，日期，布尔值或 `not_analyzed` 的字符串

```json
{"term": {"age":    26}}
{"term": {"date":   "2022-02-02"}}
{"term": {"public":    true}}
{"term": {"tag":    "full_text"}}
```

用例

```json
{
    "query" : {
        "term" : {
            "play_year" : 10
        }
    }
}
```

#### terms 查询

相比 `term` 查询，`terms` 查询可以查询 满足某个 term 的多个值

请求体的一个例子为：

```json
{
    "query": {
        "terms": {
            "play_year": [
                12,
                10
            ]
        }
    }
}
```

#### range 查询

```json
{
    "query": {
        "range" : {
            "play_year" : {
                "gte" : 10,
                "lt": 20
            }
        }
    }
}
```

范围操作符有：

- `gt` : 大于
- `gte` : 大于等于
- `lt` : 小于
- `lte` : 小于等于

#### exists 查询

```json
{
    "query": {
        "exists" : {
            "field" : "jerse_no"
        }
    }
}
```

用于查询当前文档中包含该字段的文档

#### match 查询

```json
{
    "query": {
        "match" : {
            "play_year" : "10"
        }
    }
}
```

不管是全文查询还是精确查询基本上都要用到它。无论这个字段是结构化的数据还是非结构化的数据，都可以使用 `match` 进行查询。在多次搜索中，我们需要 specify 到底我们希望返回的是多词的交还是多词的并，则需要加入 `"operator"`，如下：

```json
{
    "query": {
        "match" : {
            "team_name" : {
                 "query" : "Anthony Davids"
                 "operator" : "or"
            }
        }
    }
}
```

多词查询实际上不会选取 `AND` 和 `OR` 这种极端，更可能是使用一个相似度得分去查询数据。通过 `minimum_should_match` 来制定匹配度。

```json
{
    "query" : {
        "match" : {
            "name" : {
                "query" : "Anthony Davids",
                "minimum_should_match" : "90%"
            }
        }
    }
}
```

#### bool 查询

bool 查询是用于合并多个查询结果的布尔逻辑的查询，其主要关键字如下：

```json
{
    "bool" : {
        "must" : {"term" : {"folder" : "inbox"}}, // 可以理解为 and
        "must_not" : {"term" : {"tag" : "spam"}}, // 可以理解为 not
        "should" : [                              // 可以理解为 or
            {"term" : {"starred" : true}},
            {"term" : {"unread" : true}}
        ]
    }
}
```

距离如下

```json
{
    "query": {
        "bool": {
            "must_not": {
                "match": {
                    "play_year": 10
                }
            },
            "must": {
                "match": {
                    "team_name": "Los"
                }
            }
        }
    }
}
```

### 过滤查询

前面讲到结构化查询， 而 ES 其实还支持过滤查询，如 term、range、match 等

一个例子如下

```json
{
    "query": {
        "bool": {
            "filter" : {
                "term" : {
                    "play_year" : 15
                }
            }
        }
    }
}
```

那这个例子与结构化查询之间有什么区别呢？

- 一个过滤语句会询问每个文档的字段值，去判断是否包含着特定值

- 查询语句则会询问每个文档的字段值与特定值的匹配程度。也就是会计算每个文档与查询语句的相关性，给出一个相关性评分 `_score`，然后再按照相关性对匹配的文档进行排序，这种评分方式适用于一个没有完全配置结果的全文本搜索。

- 一个简单的文档列表，快速匹配运算并存入内存是十分方便的。这些缓存的过滤结果集合或许请求的结合使用非常高效。

- 查询结果不仅要查找相匹配的文档，还计算每个文档的相关性，所以一般来说查询语句更耗时，且不缓存结果。

所以，当我们做精确匹配搜索时，最好用过滤语句，这样可以缓存数据，且不必计算相关性。

### Reference

[Removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)

---
title: ElasticSearch (VI)
date: 2022-02-03 19:38:36
tags:
- ElasticSearch
- Apache
- Kibana
password: 0831
categories:
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch-VI/kibana_cover.png
---

### Reference

[Kibana Discover数据查询](https://www.tizi365.com/archives/796.html)
[Lucene查询语法汇总](https://www.cnblogs.com/chenqionghe/p/12501218.html)
[Kibana详细入门教程](https://www.cnblogs.com/chenqionghe/p/12503181.html)

### Kibana 介绍

Kibana 是一种辅助 ElasticSearch 进行可视化数据分析的工具，它的存在大大方便了我们对线上搜索日志数据或者机器性能等的 Inspection。可以简单得通过 docker-compose 启动一个 kibana 容器，并与 elastic 服务相连。

```json
version: "3.8"

services:
  elasticsearch:
    image: elasticsearch:7.16.2
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # - cluster.initial_master_nodes=elasticsearch
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
    networks:
      - es-net

  kibana:
    image: kibana:7.16.2
    container_name: kibana_1
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - I18N_LOCALE=zh-CN
      - xpack.monitoring.ui.container.elasticsearch.enabled=false
    volumes:
      - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:rw
    networks:
      - es-net


networks:
  es-net:
```

注意在上面的配置中，我们使用 networks 连接，并将这个 network 取名为 es-net。如果这个 es-net 不存在，则使用上述配置，否则，network 项需要改成

```json
networks:
  es-net: 
    driver: true
```

另外 kibana 环境在 docker-compose 中无法切换成中文，需要进入 `/usr/share/kibana/config` 目录中，修改或添加下列几项到 `kibana.yml` 文件中

```yaml
# Default Kibana configuration for docker target
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ] # 与 elasticsearch 中设定的名字有关
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: zh-CN
```

### Kibana 使用简单入门

当我们打开浏览器，连接 `http://127.0.0.1:5601/` 出现如下网页或者让我们添加集成时，就显式我们已经成功：

![kibana_welcome.png](https://jason24-zeng.github.io/img/ElasticSearch-VI/kibana_welcome.png)

或者我们可以使用其开发工具输入 `GET _search`查看配置，便能看到相应的返回结果如下

![discover_devops.png](https://jason24-zeng.github.io/img/ElasticSearch-VI/discover_devops.png)

### Discover Overview

我们在使用 Kibana 时，最常使用的应该就是 Kibana 的 Discover 功能，他为我们提供了一个可视化过滤与汇总统计数据的方式，是我们使用 ElasticSearch 阶段不可缺少的工具。

首先在左侧选择栏中找到 「Management（管理）: Stack Management」>> 「Kibana: Index Pattern（索引模式）」中点击创建索引模式进行创建。这里就不单独介绍了，直接使用样例进行创建，创建后在索引模式中展现如下。

![kibana_index_pattern.png](https://jason24-zeng.github.io/img/ElasticSearch-VI/kibana_index_pattern.png)

可以看到左侧选择栏有 「Analytics : Discover」这一栏，点击它，便进入了进行可视化数据统计的入口如下页面展示，接下来主要介绍这四个区域。

![discover_overview.png](https://jason24-zeng.github.io/img/ElasticSearch-VI/discover_overview.png)

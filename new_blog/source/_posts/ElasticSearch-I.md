---
title: ElasticSearch (I)
date: 2022-01-29 16:09:51
tags: 
- ElasticSearch
- Apache
categories:
- ElasticSearch
cover: /img/ElasticSearch/elastic_search_top.png
top_img: /img/ElasticSearch/elastic_search_top.png
---

## ELK Intro

ELK 实际上是三款软件的简称，分别是 Elasticsearch、Logstash、Kibana 组成，在发展过程中，又有新成员 Beats 的加入，所以就形成了 Elastic Stack。

ELK 主要是用于做日志分析的软件。其中，Elasticsearch 主要是核心存储和检索引擎，而 Kibana 则是用于将 Elasticsearch 的数据可视化化，logstash 是一个高吞吐量数据处理引擎，其将事件日志等通过 Parse 和 Transform 的方式处理后传给 Elasticsearch。而 Beats 则用于采集一切数据，再将这些数据创给 LogStash 或直接传给 Elasticsearch。整个流程框图可简化为：

![elastic_stack_workflow.png](https://jason24-zeng.github.io/img/ElasticSearch/elastic_stack_workflow.png)

上面初步讲解了四个软件之间是如何协调合作的，下面先简单定义一下这四个软件：

- **Elasticsearch**: 基于 java，是一个开源分布式搜索引擎，其特点是：分布式、零配置、自动发现、索引自动分片、索引副本机制、restful 风格接口，多数据源、自动搜索负载等。

- **Logstash**：基于 Java，是一个开源的用于收集、分析和存储日志的工具

- **Kibana**：基于 nodejs，可以为 Logstash 和 Elasticsearch 提供日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志

- **Beats**：elastic 公司开源的采集系统监控数据的代理 agent，是被监控服务器上以客户端形式运行的数据收集器的总称。由如下组成：
  
  - Packetbeat: 网络数据包分析器，用于监控、收集网络流量信息。支持 ICMP(v4 and v6)、DNS、HTTP、MySQL、PostgreSQL、Redis、MongoDB 等协议
  
  - Filebeat：用于监控、收集服务器日志文件，取代 logstash forwarder
  
  - Metricbeat：定期获取外部系统的监控指标信息，可以监控、收集 Apache、HAproxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、Zookeeper 等服务
  
  - Winlogbeat： 监控收集 Windows 系统的日志信息

## Introduction to ElasticSearch

ElasticSearch 是一种开源的实时搜索引擎，其是基于 Apache Lucene(TM) 开发的。被认为是目前最先进、功能最齐全的搜索引擎库。需要注意的是 Lucene 只是一个库，需要使用 Java 并将其集成到应用中。要想明白其工作原理，我们还需要充分的了解检索相关的知识才行。

而 Elasticsearch 便是一个使用 Java 编写的使用 Lucene 建立索引并实现搜索功能的一种引擎。它隐藏了 Lucene 的复杂性，而是让程序员能简单得通过 RESTful API 包装调用。

 除 Lucene 和全文搜索引擎外，ES 还提供了：

- 分布式的实时文件存储，每个字段都被索引并可被搜索。

- 实时分析的分布式搜索引擎

- 可扩展性，可扩展至上百台服务器，处理 PB 级的结构化或非结构化数据

而所有的这些功能，都被集中到一台服务器，通过 RESTful API 以及各种语言的 client stub 轻松进行交互。它的另一个特点就是极易上手，隐藏了复杂的搜索引擎理论。

### ES 核心概念

#### Near Real-Time 近实时 NRT

搜索平台近实时意味着我们从对文档构建索引到文档能被搜索到之间的时延很短，通常是秒级的。

#### Node 节点

一个节点可以理解成 Elasticsearch 中的一个服务器，是整个 ES 集群中的一部分，它存储数据，并参与集群的索引与搜索。一个节点需要一个名字作为标识，这个名字会是随机漫威角色的名字，在启动的时候赋予。这个名字需要额外关注，因为我们会需要确定网络中的服务器对应的节点。

通过配置集群名，一个节点被加入指定集群(default 情况下是名为 elasticsearch 的集群)。一个集群可以拥有无指定上限个节点。如果当前网络中没有任何 Elasticsearch 节点，这时启动一个节点，会默认创建并加入一个叫做 elasticsearch 的集群。

#### Cluster 集群

一个集群就是有一个或多个节点组成的，共同提供整个数据，并一起提供索引和搜索功能的抽象。一个集群会有唯一的名字标识，默认为 elasticsearch。

Note: 集群名很重要，节点需要制定集群名，才能加入该集群。所以才产品环境中，通常需要显式设定该集群名

#### Index 索引

一个索引可以认为是根据某一些特征将相似文档分到一起后的集合。可以类比关系型数据库 Database。比如，产品类目的索引，订单数据的索引。一个索引需要一个名字(小写字母) 来表示。当我们要对索引中文档进行索引、搜索、更新和删除时，都需要这个名字。

#### Type 类型

类型类似于关系型数据库中 Table 的概念。在一个索引中，可以定义一种或多种类型。一个类型可以理解为索引中的逻辑分类/分区。通常会为具有一组共同字段的文档定义一个类型。

#### Document 文档

文档是可被索引的基础信息单元。比如，一个客户的文档，一个商品的所有信息。文档以 JSON (JavaScript Object Notation) 格式表示。

在一个 index / type 里，我们可以存储任意多的文档。需要注意的是，文档虽然物理上是存在于索引之中，但文档必须索引、赋予一个索引的 type。类似于关系型数据库中 Record 的概念。除了用户定义的数据外，文档还需要包括 `_index`，`_type` 与 `_id` 字段

#### Shard & Replicas 分片与复制

有时候一个索引存储的数据超过了任意一个节点对应硬件的负荷要求，比如磁盘空间，或者 CPU 处理相应时延等。ElasticSearch 的解决方法便是分片，将索引划分成多份。每个分片本身也是功能完善并且独立的 "索引"，可被分配给任意节点。

分片的优势：

1. 提升扩展性，允许扩展内容容量

2. 分布式并行操作，提高性能/吞吐量

除此以外，ES 还管理了分片分布以及文档索引聚合的工作。

另一个问题是，当网络/云环境中，失败发生是很频繁的事情，这可能导致某个节点突然无法工作，需要一个故障转移机制去 back up 这种情况。而 ElasticSearch 的解决方法是允许创建分片的一份或多份拷贝，这个操作被称为 Replica / 复制。

复制的优势：

1. 分片/节点失败时，可以从其他分片/节点索引，提高引擎可用性。也因为这个原因，复制的分片不能与原分片处于同一节点上

2. 扩展了搜索量/吞吐量。

需要注意：分片与复制的数量可以在索引创建的时候指定。但索引一旦创建，分片数就固定了，我们只能动态改变复制的数量。而一个索引的多个分片可以存放在集群中的一台主机，也可以存在多台主机上，主要取决于集群机器数量。主分片和复制分片的具体位置被 ES 内在的策略决定。

#### ES 版本选择

ES 5.0 之前， Elastic Stack 的各个版本都不同意，容易出现版本号混乱问题。从 5.0 开始，所有的 Elastic Stack 中的项目全部统一版本号。方便大家维护和更新。

### elasticsearch 下载和安装

#### 安装 docker

详情可参考 [Docker Intro (I)](https://jason24-zeng.github.io/2022/01/29/Docker-Intro-I/)，首先安装 Docker，后续 ES，kibana 的使用都在 docker 里运行。

#### 下载 ES 和 Kibana

先运行 

```shell
docker search elasticsearch
```

查看 docker hub 公用镜像中存在的 elasticseaerch 镜像。可看到显示如下：

![docker_search_elasticsearch.png](https://jason24-zeng.github.io/img/ElasticSearch/docker_search_elasticsearch.png)

可以考虑 pull elasticsearch-kibana 镜像，使用下面指令

```shell
docker pull nshou/elasticsearch-kibana
```

或者考虑分别下载 es 和 kibana

```shell
docker pull elasticsearch:7.16.2
docker pull kibana:7.16.2
```

安装时可以看到如下的信息提示：

![docker_id.png](https://jason24-zeng.github.io/img/ElasticSearch/docker_id.png)

可以看到 docker 的镜像 ID

##### 检查机内已安装镜像

通过下面指令，可以看本机已安装的镜像

```shell
docker images
```

或者通过 docker desktop 可视化界面查看。

如图，可以看到相关的 tag， IMAGE ID 以及创造时间

![image_on_disk.png](https://jason24-zeng.github.io/img/ElasticSearch/image_on_disk.png)

##### 运行镜像

需要映射容器和本机端口 port 9200, 9300, 5601(kibana 专用)

指令为 

```shell
docker run -d -p 9200:9200 -p 9300:9300 -p 5601:5601 --name eskibana nshou/elasticsearch-kibana
```

#### Docker 内执行

##### 查看当前运行容器

指令： 

- `docker ps` （查看正在运行的容器）

- `docker ps -a` （查看所有容器）

如下图，可以看到 CONTAINER ID 为 f9c23e8de222，NAMES 指定为 eskibana

![docker_ps.png](https://jason24-zeng.github.io/img/ElasticSearch/docker_ps.png)

进入容器指令： `docker exec -it f9c23e8de222 /bin/bash`

后续的许多操作都是在该容器中执行的。

如果不存在 bash，可考虑下面指令 `docker exec -it f9c23e8de222 sh`

退出容器，则是 `cmd + d` 或者输入 `exit`

##### 启动一个已退出的容器

指令 `docker start container-id`比如上一节的 `docker stop f9c23e8de222`

##### 停止容器

指令: `docker stop container-id`

##### 删除容器

指令: `docker rm container-id`

##### 移除正在运行的容器

指令：`docker rm -f webserver`

##### 列出本地镜像

指令：`docker list`

##### 后台运行

在大部分场景下，我们希望 docker 的服务是后台运行的，可以通过 `-d` 执行容器的运行模式。比如下式，启动 `elasticsearch:7.16.2` 镜像，但是不进入容器，如果不加 `-d`，默认会直接进入 docker

```shell
docker run -itd elasticsearch:7.16.2 /bin/bash
```

##### 重命名容器

执行指令: `docker rename origin-Names new-Names`

如下图，将一个 Names 为 clever_mcnulty 的容器重命名为 abc

![docker_rename.png](https://jason24-zeng.github.io/img/ElasticSearch/docker_rename.png)

### 总结

这一章节主要简单介绍了一下 Elastic Stack 以及 Elasticsearch，并介绍了如何在 docker 中安装和运行容器 elasticsearch 与 kibana。

### Reference

[Elasticsearch入门，这一篇就够了](https://www.cnblogs.com/sunsky303/p/9438737.html)

[如何系统学习ElasticSearch：死磕 Elasticsearch 方法论（初学者必看](https://zhuanlan.zhihu.com/p/135939591)

[Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

[Mac中docker版本的ElasticSearch和Kibana安装及操作]([Mac中docker版本的ElasticSearch和Kibana安装及操作_十步杀一人-千里不留行-CSDN博客_docker elasticsearch mac](https://blog.csdn.net/m0_37609579/article/details/82698173))

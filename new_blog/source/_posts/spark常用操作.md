---
title: pyspark常用操作
date: 2022-05-13 10:37:18
tags:
- spark
- pyspark
categories:
- DA Tools
- SQL
---

## 前言

最近一周主要在做一些数据构造的工作，主要原因是大维度 embedding 的引入，造成了测试集构造数据处理流程的文件 load 量与内存占用爆发式增加，导致本地数据处理几乎不可能完成。在当前数据中，对每一个 keyword 与 video_id 对，我们需要新添加 5 个 256 维的 emb，每一行的数据几乎膨胀了 20 倍，导致在本地 join 成为 mission impossible。为了实现这样的工作，我们希冀于借助集群分布式处理的方式，使表间 join 成为可能，同时得益于内存处理，加速整个流程。

在整个过程中，也从 0 到 1 得学习了一下 pyspark 的基础知识（此文不赘述）。这篇博客主要关注的方向是平时在使用 pyspark 时常碰到的问题，以及一些解决/优化方法。

## spark-shell 操作

在刚开始接触 pyspark 的时候，我们更多的不是通过执行文件，而是一行行的代码去调用 spark API，这样的方式，不仅方便我们了解各个接口的属性，也能方便我们 debug 信息。而使用这种方式的基础，就是启用 spark-shell。

### 启动 pyspark

查看当前下载的 pyspark 路径

```shell
which pyspark
/usr/share/spark/bin/pyspark
```

启动 pyspark，并设定一些参数，确保使用内存有限制，从而不影响其他用户的使用。

```shell
#!/bin/bash
export HADOOP_USER_NAME=szci_ci_search
export HADOOP_USER_RPCPASSWORD=xxx


pyspark \
    --master yarn \
    --num-executors 200 \
    --driver-memory 10G \
    --driver-cores 2 \
    --executor-memory 8G \
    --executor-cores 4 \
    --name "xxx.xxx-spark-x" \
    --queue ci-search \
    --conf spark.speculation=true \
```

参数设置含义

```shell
num-executors: 
driver-memory:
driver-cores:
executor-memory:
executor-cores:
name:
queue:
conf:
```

启动之后，我们会得到一个 applicationID，并且 spark 会将相关环境打包上传，保证 applicationID 下的正常运行。启动界面如下

```shell
[GCC 7.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
22/05/13 11:52:42 WARN Utils: Service 'SparkUI' could not bind on port 4040. Attempting port 4041.
22/05/13 11:52:42 WARN Utils: Service 'SparkUI' could not bind on port 4041. Attempting port 4042.
22/05/13 11:52:44 INFO Client: Verifying our application has not requested more than the maximum memory capability of the cluster (102400 MB per container)
22/05/13 11:52:44 INFO Client: Will allocate AM container, with 6758 MB memory including 614 MB overhead
22/05/13 11:52:44 INFO Client: Setting up container launch context for our AM
22/05/13 11:52:44 INFO Client: Setting up the launch environment for our AM container
22/05/13 11:52:44 INFO Client: Preparing resources for our AM container
22/05/13 11:52:46 INFO Client: Uploading resource hdfs://tl3/packages/jars/spark-v2.4.7-sdi-028.tar.gz -> hdfs://tl7/user/spark/staging/.sparkStaging/application_1649942865376_6398197/spark-v2.4.7-sdi-028.tar.gz
22/05/13 11:52:49 INFO Client: Uploading resource file:/usr/share/spark/python/lib/pyspark.zip -> hdfs://tl7/user/spark/staging/.sparkStaging/application_1649942865376_6398197/pyspark.zip
22/05/13 11:52:50 INFO Client: Uploading resource file:/usr/share/spark/python/lib/py4j-0.10.7-src.zip -> hdfs://tl7/user/spark/staging/.sparkStaging/application_1649942865376_6398197/py4j-0.10.7-src.zip
22/05/13 11:52:50 INFO Client: Uploading resource file:/hadoop/spark/sparklocaldir/spark-bbd4bfd8-6578-4da3-8ee3-80511df8a90b/__spark_conf__5160579084130103039.zip -> hdfs://tl7/user/spark/staging/.sparkStaging/application_1649942865376_6398197/__spark_conf__.zip
22/05/13 11:52:52 INFO Client: Submitting application application_1649942865376_6398197 to ResourceManager
22/05/13 11:52:57 INFO Client: Application report for application_1649942865376_6398197 (state: ACCEPTED)
22/05/13 11:52:57 INFO Client: 
         client token: N/A
         diagnostics: AM container is launched, waiting for AM container to Register with RM
         ApplicationMaster host: N/A
         ApplicationMaster RPC port: -1
         queue: ci-search
         start time: 1652413972393
         final status: UNDEFINED
         tracking URL: http://keyhole.data-infra.shopee.io/keyhole/proxy?applicationId=application_1649942865376_6398197
         user: szci_ci_search
22/05/13 11:53:00 INFO Client: Application report for application_1649942865376_6398197 (state: ACCEPTED)
22/05/13 11:53:03 INFO Client: Application report for application_1649942865376_6398197 (state: ACCEPTED)
22/05/13 11:53:07 INFO Client: Application report for application_1649942865376_6398197 (state: RUNNING)
22/05/13 11:53:07 INFO Client: 
         client token: N/A
         diagnostics: N/A
         ApplicationMaster host: 10.130.80.227
         ApplicationMaster RPC port: -1
         queue: ci-search
         start time: 1652413972393
         final status: UNDEFINED
         tracking URL: http://keyhole.data-infra.shopee.io/keyhole/proxy?applicationId=application_1649942865376_6398197
         user: szci_ci_search
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.7-sdi-028
      /_/

Using Python version 2.7.16 (default, Mar 14 2019 21:00:58)
SparkSession available as 'spark'.
>>> 
```

后面我们就可以像执行 python 一样执行 pyspark 了。

## spark-submit 执行 spark 脚本

```python
spark = SparkSession.builder().master("local[1]")
          .appName("SparkByExamples.com")
          .getOrCreate()
```

## pyspark 相关常用语句

### 读取 csv 文件 （data_frame 方法）

#### 基本语句

```python
# 1.
data_frame = spark.read.csv(filename, sep= r'\t', header=True)

# 2. Using Header Record For Column Names
data_frame = spark.read.option("header", True).option(xxx, xxx).csv(filename)


# 3.
df = spark.read.format("csv")
                  .load(filename)
# or
df = spark.read.format("org.apache.spark.sql.csv")
                  .load(filename)
```

#### Extension 1.  读取多个文件

```shell
data_frame = spark.read.csv("path1,path2,path3") # 这个我测试的时候没法成功

file_conf = [filename1, filename2, filename3]
data_frame = spark.read.csv(file_conf)
```

#### Extension 2. 读取多个文件

```shell
data_frame = spark.read.csv(Foldername)
```

### 读取 csv 文件 （RDD 方法, 用到 sparkContext-sc)

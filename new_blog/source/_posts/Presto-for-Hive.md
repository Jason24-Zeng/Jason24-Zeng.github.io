---
title: Presto for Hive
date: 2022-02-28 15:49:36
tags: 
- Presto
- Hive
- HDFS
categories:
- DA Tools
- SQL
cover: /img/Presto-for-Hive/Presto-logo.jpeg
top_img: /img/Docker-Intro-I/Presto-logo.jpeg
---

## 什么是 Presto？

Presto 是一种开源的分布式 SQL 查询引擎，它能面向大范围 size 的数据 (从 GB 到 PB)进行交互式的数据分析。它能从不同数据源中取数据，包括 Hive, Cassandra，关联性数据库等。它最大的特点就是使用内存分布式运行，处理数据反应比 hive 都快多个数量级。

### Presto 基本语句

#### 创建并写入表 I

基本的建表语句为

```sql
CREATE TABLE [ IF NOT EXISTS ] table_name [ ( column_alias, ... ) ]
[ COMMENT table_comment ]
[ WITH ( property_name = expression [, ...] ) ]
AS query
[ WITH [ NO ] DATA ]
```

可以看到其实整个表的创建和 hive 几乎一致，只是对于一些 `partition` 与 `format`等操作，Presto 将他们放到了 `property_name` 中，比如 

```sql
CREATE TABLE orders (
   order_date VARCHAR,
   order_region VARCHAR,
   order_id BIGINT,
   order_info VARCHAR
) WITH (partitioned_by = ARRAY['order_date', 'order_region'])
```

需要注意的是每个 column 的类型和 hive 有一定的区别，presto 中的类型可以查询：

[Data Types &#8212; Presto 0.270 Documentation](https://prestodb.io/docs/current/language/types.html#string)

且对于 table 中某个 column 的 type，我们可以用 `typeof(column_name)` 去获取。

如果我们希望查看整个 table 的列/分区/库结构，则可以输入

```sql
SHOW COLUMNS FROM table
SHOW PARTITION FROM table
SHOW CREATE TABLE table
```

#### 创建并写入表 II

对于已创建的表，我们可以使用 Insert Into 语句写入表中。

```sql
INSERT INTO table_name [ ( column [, ... ] ) ] query
```

我们可以看到，这个表除了可以全量插入数据，还可以对特殊的列进行插入操作。其他列会变成 NULL

**Notice:** Presto 没有 Insert Overwrite 的操作，我们只能通过 Delete 然后 Insert Into 的方法写入已创建表

#### 删除表 I

直接把表连同表结构进行删除

```sql
DROP TABLE  [ IF EXISTS ] table_name
```

#### 删除表 II

对表的某个分区进行删除，或者删除整个表，但是保留表的列结构等信息。

```sql
DELETE FROM table_name [ WHERE condition ]
```

#### 对表进行修改

##### 修改表名

```sql
ALTER TABLE [ I EXISTS ] name RENAME TO new_name
```

##### 增加表列

```sql
ALTER TABLE [ IF EXISTS ] name ADD COLUMN [ IF NOT EXISTS ] column_name data_type [ COMMENT comment ] [ WITH ( property_name = expression [, ...] ) ]
```

##### 删除表列

```sql
ALTER TABLE [ IF EXISTS ] name DROP COLUMN column_name
```

##### 修改表列名

```sql
ALTER TABLE [ IF EXISTS ] name RENAME COLUMN [ IF EXISTS ] column_name TO new_column_name
```

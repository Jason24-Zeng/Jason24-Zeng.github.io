---
title: 'Pandas: General Use'
date: 2022-04-19 22:37:27
tags: 
- Python
- Pandas
- DataFrame
categories:
- Coding Language
- Python
cover: /img/python_tag.png
top_img: /img/python_tag.png
---

## 简介

日常生活中，我们经常会面临表格数据的处理，而 pandas 是一个封装较好的 API，我们经常可以借助于 pandas 更方便得处理数据。这片文章就简单记录一下工作中常使用的 pandas 语句

## IO 输入输出

### 读取

```python
# csv 文件读取
pd.read_csv(filename, sep = '\t')
```

### 写入

```python
pd.to_csv(filename, sep = '\t', index = False, header = False)
```

## 选取

### 根据某一列聚合遍历

```python
# 已知 data_frame
data_frame
# 固定列名
col_name_list = ["feat_1", "feat_2"]
data_frame.columns = col_name_list
# 聚合某一列
data_group = data_frame.groupby("feat_1")
# 遍历每一个根据 feat_1 聚合的 data_frame
for name, group in data_group:
    # 遍历 group 这个 dataframe
    print("The row of dataframe is as follow when feat_1 equal to {}".format(name))
    for row_index, row in group.iterrows():
        feat_2_val = row["feat_2"]
        print("row_index : {}, feat_2_val : {}".format(row_index, feat_2_val))
# 查看聚合的所有 feat_1 的值，去重后结果
print(data_group.groups.keys())
# 根据 feat_1 的值拿出其中一个 group 的 data_frame
data_group.get_group(feat_1_val)
```

### 笛卡尔积式 Cross Join

```python
# label 为 3 的 dataframe
df_label_1
# label 为 2 的 dataframe
df_label_2
# label 为 1 的 dataframe
df_label_3
# 初始化最终的 data_frame
data_frame_cross = pd.DataFrame()
# merge 两两 cross join 的结果
data_frame_cross = pd.concat([data_frame_cross, pd.merge(df_label_1, df_label_2, how = 'cross')])
data_frame_cross = pd.concat([data_frame_cross, pd.merge(df_label_2, df_label_3, how = 'cross')])
data_frame_cross = pd.concat([data_frame_cross, pd.merge(df_label_1, df_label_3, how = 'cross')])
# 重新排序
data_frame_cross.index = range(data_frame_cross.shape[0])
```

### 某一列的所有取值

```python
# Series 化 unique
pd.Series([2, 1, 3, 3], name='A').unique()
```

### 根据 bin 分桶

`pd.cut` 注意参数 `bins`，`right`, `labels`

### 对 groupby 的 dataframe 重新设置 index

```python
df=df.groupby('A').apply(lambda x: x.reset_index(drop=True)).drop('A',axis=1).reset_index()
grouped.get_group('foo').iloc[0:3]
```

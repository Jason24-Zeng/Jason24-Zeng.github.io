---
title: MMU - 视频关键帧与商品识别
date: 2023-03-19 22:37:39
tags: 
- Multi Media Understanding
- 视频抽帧
categories:
- Business MindMap
- Search
password: 0418
cover: /img/MMU-视频关键帧与商品识别/entire_workflow.png
top_img: /img/MMU-视频关键帧与商品识别/entire_workflow.png
---

## 背景

当前对TikTok视频建设了TT热门视频供给、TT视频内置挂载商品关系供给、TT seller视频供给等项目，对TikTok视频已有稳定的各方式供给，为了进一步提升站内带货视频供给，需要开辟新的供给视频源，根据产品的预研，选定了Instagram APP（简称Ins），

- 业务BRD： [[BRD] Instagram 视频爬取需求](https://docs.google.com/document/d/1bTo6msrqHpRuPj6WgcS_zEmzT44yT9VDOaF2GLxAaRw/edit#heading=h.6xkm8aap3tue)

- 需求PRD：https://confluence.shopee.io/pages/viewpage.action?pageId=1618316343

- MMU需求：基于对爬取Ins视频检测其中关键帧，并识别其对应Shopee的商品，实现Ins视频挂货。

## 整体框架

![entire_workflow_framework](https://jason24-zeng.github.io/img/MMU-视频关键帧与商品识别/entire_workflow.png)

## 关键帧检测

![keyframe_detection_framework](https://jason24-zeng.github.io/img/MMU-视频关键帧与商品识别/keyframe_detection.png)

视频帧：抽帧策略为1s1帧；

Conditioned Clustering: 商品ROI做聚类，并加入以下限制条件：

- 同一类的ROI时间跨度不超过α；

- 同一类的ROI长宽比相似度不超过β；

后处理：经过聚类后，过滤掉不包含商品的视频帧，并且视频被切分为若干short clips, 每个clip中选取关键帧，选取原则：

- 关键帧尽可能从不同角度展示商品；

- 商品尽可能位于视频中心位置；

- 过滤掉视频caption和关键帧的相关性较低的帧；

备注：关键帧检测环节会缓存商品检测和特征提取结果，同时会输出关键帧中包含的商品的bbox和ROI特征。

1. http://34.126.107.210:8004/liujianwei/projects/ins_video_recognition/outputs/pos_keyframe.html

2. http://34.126.107.210:8004/liujianwei/projects/ins_video_recognition/outputs/neg_keyframe.html

## 商品检索

### 一期方案：图像检索

![image_retriever](https://jason24-zeng.github.io/img/MMU-视频关键帧与商品识别/image_retrieve_workflow.png)

#### 检索库

- 数据规模（ID区）

- 总商品库： 2254956845

- 近30天销量大于0： 17248311

- 近90天销量大于0： 29211612

- 历史销量大于0： 55947101

- 索引库：历史销量大于0商品5.5kw

- 商品数：~5.5kw

- 图片数：~28kw

- 特征数：~16kw

#### 检索性能

- 压测数据

- Faiss压测：https://docs.google.com/spreadsheets/d/1xZzc5HljVe9AIk04FDKurGXxh1_C6c0lJXdY4ZNg9Zw/edit#gid=877678075 

- Milvus压测：https://docs.google.com/spreadsheets/d/1xZzc5HljVe9AIk04FDKurGXxh1_C6c0lJXdY4ZNg9Zw/edit#gid=314759578 

- Vespa压测：https://docs.google.com/spreadsheets/d/1xZzc5HljVe9AIk04FDKurGXxh1_C6c0lJXdY4ZNg9Zw/edit#gid=1130775177 

- Hawing压测：https://docs.google.com/spreadsheets/d/1xZzc5HljVe9AIk04FDKurGXxh1_C6c0lJXdY4ZNg9Zw/edit#gid=1527588015 

- 全链路时延：~60ms

- 商品检测：20ms

- 特征提取：8ms

- milvus检索：20ms

#### 算法方案： [MMU-商品检索TechDesign（新）](https://docs.google.com/document/d/19b2JNBm7LrthAed5FKOFcggWfXDP9ByCwpnRMxsl6Vo/edit#heading=h.izle0ilc70gb)

#### 二期方案：多模态检索

![mmu_retrieve](https://jason24-zeng.github.io/img/MMU-视频关键帧与商品识别/mmu_retrieve_workflow.png)

#### 算法方案：

![mmu_retrieve_algo](https://jason24-zeng.github.io/img/MMU-视频关键帧与商品识别/mmu_retrieve_workflow_algo.png)

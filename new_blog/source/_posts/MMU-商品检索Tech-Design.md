---
title: MMU - 商品检索Tech Design
date: 2023-03-19 22:49:05
tags: 
- Multi Media Understanding
- 商品检索
categories:
- Business MindMap
- Search
password: 0418
cover: /img/MMU-商品检索Tech-Design/mmu_item_retrieve.png
top_img: /img/MMU-商品检索Tech-Design/mmu_item_retrieve.png
---

# 项目背景

## Livestreaming场景

- [直播回放切片](https://confluence.shopee.io/pages/viewpage.action?pageId=1393807770)：获取直播回放，算法切分成直播切片，每个切片对应一个商品，供直播业务及相关业务使用。

- [实时讲解识别](https://confluence.shopee.io/pages/viewpage.action?pageId=1428801053)：实时识别直播间中正讲解商品，支持内容进电商。

- 带货直播间[推荐](https://confluence.shopee.io/pages/viewpage.action?pageId=1089486116)：根据商品的封面图进行相似商品推荐，推荐相似商品或直播间，提升用户体验和推荐核心指标。

## Video场景

- 商品视频相关性（小黄车）

- 视频检索商品

- TT商品检索Shopee商品

## Marketplace场景

- 电商竞品分析：建立初版竞对情报分析系统，推动电商业务

## Shopee图搜场景

主要指标：

- 检索库规模：

- 响应时耗：

# 项目目标

支持图片、item粒度商品检索，检索库 ROI 规模百亿

初期目标：

- 输出完整的端到端 demo 效果

- 规模：1亿 & 50亿

- 速度：端到端 1s 以内

- 精度：优于图搜

### 细粒度商品检索

- 单模型商品检索（一期）
  
  - 主体商品检测
  
  - 单库商品检索

- 多模型商品检索（二期）
  
  - 多类商品检测
  
  - 商品类目预测
  
  - 分库商品检索

- 完善的商品检索系统（三期）
  
  - open-world 商品检测
  
  - L1~L5 商品分类
  
  - 商品信息抽取

### 跨模态商品检索

- 单模态商品特征/相关性（一期）
  
  - Image→Image：竞品分析、TT商品检索Shopee商品       
  
  - Text→Image：主体选框
  
  - Video→Image：小黄车

- 多模态商品特征/相关性（二期）

# 方案调研

## [细粒度商品检索](https://github.com/willard-yuan/awesome-cbir-papers)

### [拍立淘](https://developer.aliyun.com/article/161333)（[paper](https://arxiv.org/pdf/2102.04674.pdf)）

![pailitao](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/pailitao.png)

### [微信扫一扫](https://mp.weixin.qq.com/s/fiUUkT7hyJwXmAGQ1kMcqQ)

![wechatsaoyisao](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/wechatsaoyisao.png)

### [EBay](https://arxiv.org/abs/1706.03154)

![Ebay](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/ebay.png)

### [百度识图](https://zhuanlan.zhihu.com/p/22451308)

- 商品主体检测：用于自动定位用户感兴趣的商品，去除背景、多主体等因素的影响，也有利于抽取的语义特征的对齐；

- 商品品类识别： 通过识别商品的主体的品类， 使得在检索时可以在商品子数据子库进行搜索，提升检索的效果与效率；

- 商品特征表示： 通过学习获得商品主体的判别性特征， 使得同款商品距离更近且非同款商品相距更远； 对光照、姿态、遮挡等变化有一定的鲁棒性

## 跨模态商品检索

### [M5Prodcuts](https://xiaodongsuper.github.io/M5Product_dataset/index.html)（阿里）

![m5product1](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/m5product.png)

- 模态重要性排序：Text > Image > Table > Video > Audio

![m5product_metrics](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/m5product_metrics.png)

# 方案设计

## 通用商品检索

### 整体框架

### 应用框架

- 通用场景

![general_scenario](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/general_scenario.png)

- 直播场景

![livestreaming_scenario](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/livestreaming_scenario.png)

### 商品检测

![item_detection](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/item_detection.png)

- 数据集

| 数据库         | Concepts | Images | bbox num |
| ----------- | -------- | ------ | -------- |
| CoCo        | 80       | 118K   | 1M       |
| DeepFashion | 1050     | 123K   | -        |
| WAB         | -        | 1.0M   | 1.6M     |
| Object365   | 365      | 609K   | 10M      |
| OpenImages  | 600      | 2M     | 14M      |
| Bamboo-DET  | 809      | 3M     | 27M      |

- 类目体系（微信扫一扫）

![cate_system_wechat](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/cate_system_wechat.png)

- [优化方向](https://arxiv.org/pdf/2103.02603.pdf)
  ![optimization_target](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/optimization_target.png)
  
  - Objectness/One-stage Detection
    
    - [NanoDet-Plus](https://github.com/RangiLyu/nanodet): Super fast and high accuracy lightweight anchor-free object detection model
    
    - [YOLOv7](https://github.com/WongKinYiu/yolov7): Trainable bag-of-freebies sets new state-of-the-art for real-time object detectors
    
    - [Generalized Focal Loss V2](https://openaccess.thecvf.com/content/CVPR2021/papers/Li_Generalized_Focal_Loss_V2_Learning_Reliable_Localization_Quality_Estimation_for_CVPR_2021_paper.pdf): Learning Reliable Localization Quality Estimation for Dense Object Detection
    
    - [IoU/GIoU/DIoU/CIoU Loss](https://zhuanlan.zhihu.com/p/104236411)
  
  - Incremental Learning
    
    - [iOD](https://github.com/JosephKJ/iOD): Incremental Object Detection via Meta-Learning
    
    - [OWOD](https://github.com/JosephKJ/OWOD): Towards open world object detection
    
    - [Faster ILOD](https://github.com/CanPeng123/Faster-ILOD): Incremental Learning for Object Detectors based on Faster RCNN
  
  - Open-set/One-shot Detection
    
    - [CoAE](https://github.com/timy90022/One-Shot-Object-Detection): One-Shot Object Detection with Co-Attention and Co-Excitation
    
    - [UP-DETR](https://github.com/dddzg/up-detr): Unsupervised Pre-training for Object Detection with Transformers
    
    - [Os2d](https://github.com/aosokin/os2d): One-stage one-shot object detection by matching anchor features
    
    - [Meta faster r-cnn](https://github.com/GuangxingHan/Meta-Faster-R-CNN): Towards accurate few-shot object detection with attentive feature alignment
    
    - [OWL](https://arxiv.org/pdf/2205.06230v2.pdf): Simple Open-Vocabulary Object Detection with Vision Transformers

### 检索特征

![retrieving_feature_1](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/retrieving_feature_1.png)

![retrieving_feature_2](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/retrieving_feature_2.png)

- 数据集
  
  |             | 训练集            | 验证集           | 数据集            |
  | ----------- | -------------- | ------------- | -------------- |
  | Aliproducts | 2551372/50019  | 148387/50030  | 2699759/50030  |
  | DeepFashion | 120927/16940   | 118630/16941  | 239557/33881   |
  | Inshop      | 52712/7982     | -             | 52712/7982     |
  | Products10K | 141931/9691    | 55376/9691    | 197307/9691    |
  | SOP         | 59551/11318    | 60502/11316   | 120053/22634   |
  | WAB         | 478347/35786   | 314612/25204  | 792959/60990   |
  | 合计          | 3404840/131736 | 697507/113182 | 4102347/185208 |

- [优化方向](https://arxiv.org/pdf/2003.08505.pdf)
  
  - Largemargin Loss
    
    - [CosFace](https://openaccess.thecvf.com/content_cvpr_2018/papers/Wang_CosFace_Large_Margin_CVPR_2018_paper.pdf): Large Margin Cosine Loss for Deep Face Recognition
    
    - [ArcFace](https://arxiv.org/pdf/1801.07698): Additive Angular Margin Loss for Deep Face Recognition
    
    - [CurricularFace](https://github.com/HuangYG123/CurricularFace): Adaptive Curriculum Learning Loss for Deep Face Recognition
    
    - [AdaFace](https://github.com/mk-minchul/AdaFace): Quality Adaptive Margin for Face Recognition
    
    - [MagFace](https://arxiv.org/pdf/2103.06627.pdf): A Universal Representation for Face Recognition and Quality Assessment
    
    - [CircleLoss](https://arxiv.org/pdf/2002.10857.pdf): A Unified Perspective of Pair Similarity Optimization
  
  - Rank Loss
    
    - [ROADMAP](https://github.com/elias-ramzi/ROADMAP): Robust and Decomposable Average Precision for Image Retrieval
    
    - [Recall@k Surrogate Loss](https://github.com/yash0307/RecallatK_surrogate) with Large Batches and Similarity Mixup
    
    - [SmoothAP](https://github.com/Andrew-Brown1/Smooth_AP): Smoothing the path towards large-scale image retrieval
    
    - [FastAP](https://github.com/kunhe/FastAP-metric-learning): Deep metric learning to rank
    
    - [BlackboxAP](https://openaccess.thecvf.com/content_CVPR_2020/papers/Rolinek_Optimizing_Rank-Based_Metrics_With_Blackbox_Differentiation_CVPR_2020_paper.pdf): Optimizing rank-based metrics with blackbox differentiation

### 类目预测

![cate_pred](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/cate_pred.png)

- 优化方向
  
  - [CenterLoss](https://ydwen.github.io/papers/WenECCV16.pdf): A Discriminative Feature Learning Approach for Deep Face Recognition
  
  - [Deep Cluster](https://github.com/facebookresearch/deepcluster): Deep Clustering for Unsupervised Learning of Visual Features
  
  - [SeLa](https://github.com/yukimasano/self-label): Self-labelling via simultaneous clustering and representation learning
  
  - [SwAV](https://github.com/facebookresearch/swav): Unsupervised Learning of Visual Features by Contrasting Cluster Assignments

## 跨模态商品检索

![mmu_item_retrieve](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/mmu_item_retrieve.png)

### 单模态商品特征/相关性（一期）— Inter-Modality Contrastive Learning

![single_item_retrieve](https://jason24-zeng.github.io/img/MMU-商品检索Tech-Design/single_item_retrieve.png)

### 多模态商品特征/相关性（二期）— Masked Multi-Modal Tasks

- Masked Region Prediction task (MRP)

- Masked Language Modeling task (MLM)

- Mask Entity Modeling task (MEM)

- Mask Frame Prediction task (MFP)

- Mask Audio Modeling task (MAM)

# 工程方案

## 扫一扫工程方案

1. 离线系统

2. 在线系统

# 五、项目进展

## 系统压测

### 调研图搜代码和系统现状

1、已完成

### 资源申请

1、等待SRE复工后审批(15台物理机)

### milvus压测(底层是faiss&其他检索库，叠加分布式能力&平台能力)

1、数据导入：索引建立完成

2、系统压测：等待机器资源审批通过后才能压测。

### faiss压测(作为备份方案压测)

#### Python版压测数据

https://docs.google.com/spreadsheets/d/1LLg983Fb2OFxHnvQONtcTwVkmYOMH062CbCgE2Le2Sw/edit#gid=0

#### C++版压测中

#### 数据分析：Python版在HNSW索引下表现较好(CPU使用率较高)

1. python faiss：m16，ef200，向量数 3kw，向量维度512，nq=32，avg=60ms，p99=68ms - [https://docs.google.com/spreadsheets/d/1LLg983Fb2OFxHnvQONtcTwVkmYOMH062CbCgE2Le2Sw/edit#gid=0](https://monitoring.infra.sz.shopee.io/grafana/d/awyJMKeGa/imagesearch-spex?orgId=44&refresh=30s)

2. 原图搜：1亿向量，p99=1.06s ~ 2.34s - [https://monitoring.infra.sz.shopee.io/grafana/d/awyJMKeGa/imagesearch-spex?orgId=44&refresh=30s](https://monitoring.infra.sz.shopee.io/grafana/d/awyJMKeGa/imagesearch-spex?orgId=44&refresh=30s)

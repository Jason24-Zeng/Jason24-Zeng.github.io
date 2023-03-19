---
title: Instagram 视频挂车商品
date: 2023-03-19 16:54:27
tags: 
- 视频商品相关性
categories:
- Business MindMap
- Search
password: 0418
cover: /img/Instagram-视频挂车商品/entire_workflow.png
top_img: /img/Instagram-视频挂车商品/entire_workflow.png
---

# 背景

## 基本背景

当前对TikTok视频建设了TT热门视频供给、TT视频内置挂载商品关系供给、TT seller视频供给等项目，对TikTok视频已有稳定的各方式供给，为了进一步提升站内带货视频供给，需要开辟新的供给视频源，根据产品的预研，选定了Instagram APP（简称Ins），具体产品预研及BRD： [[BRD] Instagram 视频爬取需求](https://docs.google.com/document/d/1bTo6msrqHpRuPj6WgcS_zEmzT44yT9VDOaF2GLxAaRw/edit#heading=h.6xkm8aap3tue) 。

经过梳理，整体Ins视频供给流程概貌和相关团队如下：

1. 供给团队：提供爬取信息来源，例如目标爬取种子

2. 爬虫团队：破解和爬取Ins视频及相关数据

3. Search团队：提供及实现视频-商品匹配方案

4. MMU团队：提供商品关键截帧及商品检测能力

5. 主站搜索团队：提供以图搜商品能力

6. video core：视频发布

7. 视频Search&RCMD：内容分发

另附：[Search侧需求PRD](https://confluence.shopee.io/pages/viewpage.action?pageId=1618316343)

## Ins视频数据摸底

### 爬取种子选取

根据调研，Ins的搜索能力较差，只能通过搜索hashtag或作者名实现检索，且检索结果更多的是按照时间排序，相关性排序因子较弱。

目前通过电商强相关的hashtag进行Ins检索，爬取的视频相对好一些。目前当前人工找了57个hashtag，后续正式爬取hashtag会扩充到100+个左右，扩展过程中，找hashtag的逻辑跟着57个逻辑是一样的，主要是找电商强相关hashtag，例如：tas wanita（女包）、dapur（厨房好物）、Main ananak（儿童玩具）等。

产品梳理的hashtag列表： [ins印尼#搜索结果](https://docs.google.com/spreadsheets/d/1o64Koqk8H4_OditjSkHBNyDvZN1C3RTZO3SuJgiUb_Q/edit#gid=0)

### 爬取视频初步调研

爬虫团队根据已有的 57 个hashtag，爬取了 6000 条视频（hashtag种子并发爬取，可以大概认为这 6000 视频是随机从 57 个hashtag爬的），从 6000 条视频中随机抽了 100 条视频做了人工review，有带货倾向的视频有 72%，20% 视频与商品无关，8% 视频链接无法打开。

### 爬取商品信息情况

TikTok 视频爬取各项目中，会通过**各种爬取信息破解手段**找到视频里相关商品的信息，但Instagram 视频并无带货挂车功能，经过爬虫团队调研，无法给出视频中的商品相关信息。

对于 Instagram 视频的商品匹配，只能通过全库检索及匹配方式完成，而这种方式是目前匹配方式中难度最大的一种。

### 项目初步目标

根据 Supply 需求和视频爬取情况，项目本 Q 目标为每天供给 1k~2k 带货视频。

## 基本结构

这里我们集中讨论视频-商品匹配方案的设计。

主要分下面几部分：

二、依赖能力梳理

三、视频-商品匹配方案

四、流程数据依赖与输出

# 依赖能力梳理

本章梳理了可以依赖的相关能力，并对当前已可用的能力做初步效果摸底。未 ready 能力需等能力提供方 ready 后接入整体流程再看效果。

复用第一章的 2.2 部分的 100 个case，去掉与食品相关视频（可通过 hashtag 过滤掉）和无法播放视频，剩下 86 个视频。下面已有能力通过跑这 86 个case（下面简称 “调研 case”）能力得分及结合人工标注，做下能力介绍和初步的能力摸底。

## 视频检索商品服务

Search组内之前建设过一个视频-商品检索的服务，通过视频对商品库中商品做检索，目前该服务应用于Seller视频匹配项目（项目主页：[Seller视频商品挂载项目](https://confluence.shopee.io/pages/viewpage.action?pageId=1439136118)）中，检索服务项目主页：[视频检索商品项目](https://confluence.shopee.io/pages/viewpage.action?pageId=1235039010)。

可以通过该服务做一路商品召回。

在调研case上，服务检索分数不同阈值下，结合人工标注，视频-商品相关性表现如下：

1. 当不设阈值时，有结果率：100%，top1准确率 25.6%（top2：25.6%，top3：27.9%）。

2. 设定阈值 >0.4，有结果率：83.7%，top1准确率 30.6%（top2：30.6%，top3：30.6%）。

3. 设定阈值 >0.5，有结果率：60.5%，top1准确率 38.5%（top2：38.5%，top3：40.4%）。

4. 设定阈值 >0.6，有结果率：23.3%，top1准确率 55.0%（top2：55.0%，top3：55.0%）。 

根据准确率和有结果率情况，将该路纳入召回是可行的。

该检索服务商品库范围是 订单 top200 万商品 + PV top200 万商品，去重后大概 250~300万 商品。在未召回结果的视频中，随机抽检了 10 个视频，发现视频中商品均未在商品库中。

为了对召回商品做进一步补充，会加入 主站以图搜索商品 和 mmu 商品检测 等召回能力。

## 主站以图搜商品服务

Shopee 手机 APP 上有以图搜商品的功能，根据之前试用，能有比较不错的效果，期望能够利用此能力到Ins项目爬取项目中。

根据调研，目前可以拿到的且可比较好代表视频商品的是视频封面图，我们使用主站以图搜商品服务，对 video 的封面图发起商品检索。商品库为 ID 地区历史总销量 >0 的商品，库大小为 5000 万左右。

以图搜商品返回得分不同阈值下，结合人工标注，视频-商品相关性表现如下：

1. 当不设阈值时，有结果率：100%，top1 准确率 33.3%（top2：35.9%，top3：35.9%）。

2. 设定阈值 >0.9，有结果率：97.4%，top1 准确率 34.2%（top2：36.8%，top3：36.8%）。

3. 设定阈值 >0.92，有结果率：69.2%，top1 准确率 44.4%（top2：46.3%，top3：46.3%）。

4. 设定阈值 >0.95，有结果率：21.8%，top1 准确率 76.5%（top2：82.4%，top3：82.4%）。 

根据准确率和有结果率情况，将该路纳入召回是可行的。

另外：后面会接 MMU 视频商品帧识别服务，会用识别出的带商品的帧图再次过主站以图搜商品的服务，作为另外一路召回。

## MMU 视频商品帧识别及商品识别服务

[MMU-视频商品关键帧检测与商品识别TD](https://docs.google.com/document/d/1GkUXBaMqUO54YxusUoMZkjNDTMgc_6RkN_rzyGJvuZI/edit#)

## 小黄车视频-商品相关性服务

有各路召回的 item 后，可以调用 视频 - 商品 相关性服务，得到统一可比的 item 相关性分数。

小黄车 视频-商品 相关性三期项目详见：[视频-商品相关性三期](https://confluence.shopee.io/pages/viewpage.action?pageId=1446387013)。

调研期间，有两路可用召回：1. 视频检索商品服务召回，2. 主站以图搜商品服务召回。

只是用 1. 视频检索商品服务召回时，召回 item计算小黄车视频-商品相关性分数，结合人工标注效果如下：

1. 不设阈值，有结果率：100%，准确率：29.1%。

2. 设置阈值 >0.65，有结果率：45.3%，准确率：53.8%。

3. 设置阈值 >0.7，有结果率：27.9%，准确率：79.2%。

4. 设置阈值 >0.75，有结果率：16.3%，准确率：100%。

使用 1. 视频检索商品服务召回 + 2. 主站以图搜商品服务召回 结果取并，并过小黄车服务：

1. 不设阈值，有结果率：100%，准确率：44.2%。

2. 设置阈值 >0.65，有结果率：60.5%，准确率：63.5%。

3. 设置阈值 >0.7，有结果率：37.2%，准确率：81.2%。

4. 设置阈值 >0.75，有结果率：23.3%，准确率：100%。

可以看出，由于靠谱的召回结果增多，第2路召回在有结果率和准确率上均有正向补充。

# 视频-商品匹配方案

## 上下游总体结构

整体流程分为爬虫团队爬取 Ins 视频，发送给 Search 侧做匹配，匹配完后 Search 侧将结果写入爬虫数据接收 kafka。

其中 Search 内部需要对爬取视频做触发特征计算并存 kafka，依赖的 MMU 视频帧识&商品识别模块由异步消费上面 kafka 方式给出结果到另外一个 kafka，最终 Search ALG 侧从 kafka 取出数据做后续匹配。

该流程定时每天运行，消费 kafka 中之前没有消费的所有 msg。

具体大块的各工程团队配合如下流程：

![整体流程](https://jason24-zeng.github.io/img/Instagram-视频挂车商品/entire_workflow.png)

## 匹配流程

上图中算法匹配逻辑模块包含4路召回及商品排序选择流程，具体流程如下图所示：

![算法匹配流程](https://jason24-zeng.github.io/img/Instagram-视频挂车商品/search_workflow.png)

## 商品相关性排序

各路召回商品集后，需要对商品做综合排序，最终选出相关商品，并做视频-商品关联关系输出。

目前本项目要求一个视频只挂一个商品。（调研了100个Ins视频，90%左右的视频只主卖一种商品。）

### 规则排序

拿到召回商品集后，会对每个商品调用视频相关性模型（小黄车服务），得到商品跟当前视频的相关性分数，除此以外，也会利用各召回路给出的分数参与排序。

由于各路召回来自不同检索服务，其返回结果分数之前没有可比性，为了更加可控项目初期使用规则融合公式对商品进行排序。

融合公式主体采用线性加权，但由于每路召回只有自己路服务的分数并不可比，所以通过分段加权方式将item分数做融合。

定义：

1. `l2_pair_rel_score`：小黄车服务分数

2. `recall_search_score`：Search视频检索商品得分

3. `recall_cover_score`：视频封图主站检索商品得分

4. `recall_frames_score`：视频截帧主站检索商品得分

5. `recall_videommu_score`：视频内容检测商品得分

6. `weight{*}`：各因子参数

7. `threshold{*}`：各因子阈值

公式：

```cpp
final_score = weight1 * l2_pair_rel_score + weight2 * recall_score_boost

recall_score_boost = weight11 * recall_search_norm + weight12 * recall_cover_norm + weight13 * recall_frames_norm + weight14 * recall_videommu_norm

recall_search_norm = 1.0 if recall_search_score > threshold1 else 0.1

recall_cover_norm = 1.0 if recall_cover_score > threshold2 else 0.1

recall_frames_norm = 1.0 if recall_frames_score > threshold3 else 0.1

recall_videommu_norm = 1.0 if recall_videommu_score > threshold4 else 0.1
```

### 模型排序

第一个版本上线后，项目产出的结果数据会过 QC 人审，根据人审 label 作为 label，可以尝试做融合公式的参数模型化。

上面接介绍到，各路召回结果分数之间不可比且需要非线性拟合，外加上其他一些特征，选择决策树模型（例如 XGBoost、LightGBM 等）较为合适。

由于第一版上线前没有该通路的审核结果和样本，且为了前期高可解释性以及方便调整效果，暂不用模型做分数融合，这里暂时不做展开。

# 流程数据输入与输出

整个流程为爬虫给到待匹配视频信息，Search侧返回视频-商品关联信息。整个流程通过kafka交互。

下面是爬虫侧输入数据和Search返回的数据。数据均为json字符串。

## 输入

输入kafka消息为json字符串，解开为dict结构，字段可以参照下面示例：

```json
{
  "aid": "CfWXG0jjcn3",
  "author_id": "9528671315",
  "bit_rate": 1687842,
  "caption": "#longtunikjumbo #longtunikmurah #mididress #tunikmurah #tunikjumbo #tunikhijab #tunikcantik #tunikbigsize #tunikkekinian #tunikmuslimah #tunikkatun #pakaianmuslimah #pakaianwanita #pakaianmurah #bajumuslimah #bajuwanita",
  "cover_image_mms_url": "http://img.sp.mms.shopee.sg/sg-11134113-23020-9b08pg4hdxnv58",
  "ctime": 1677233454,
  "detect_lang": "id",
  "duration": 14200,
  "hashtag": "pakaianmurah",
  "height": 1280,
  "id": 5656,
  "job_id": "20230224120004yQuLlC",
  "keyword": "",
  "l1_topic": "Beauty & Fashion",
  "l2_topic": "Dressing Outfits",
  "last_modify_time": 1656420248,
  "mms_url": "https://vod.sp-cdn.shopee.com/c3/98934353/122/A3oyOcvkAOj54QkaEdsIACY.mp4",
  "mms_vid": "sg_2bae65b8-ec69-471c-8b4f-258d77f7989b_000345",
  "mmu_item_image_list": [
    {
      "idx": 10,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-e918gf2hdxnv2e_sprite1x1-51"
    },
    {
      "idx": 3,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-e918gf2hdxnv2e_sprite1x1-16"
    },
    {
      "idx": 12,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-e918gf2hdxnv2e_sprite1x1-61"
    },
    {
      "idx": 14,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-e918gf2hdxnv2e_sprite1x1-71"
    },
    {
      "idx": 13,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-e918gf2hdxnv2e_sprite1x1-66"
    }
  ],
  "mmu_item_list": [
    {
      "item_id": 10875013561,
      "score": 0.830738215
    },
    {
      "item_id": 13170765111,
      "score": 0.806216715
    },
    {
      "item_id": 10390521602,
      "score": 0.80292648
    },
    {
      "item_id": 16583890561,
      "score": 0.80149953
    },
    {
      "item_id": 9973913969,
      "score": 0.801484525
    },
    {
      "item_id": 21443409208,
      "score": 0.799377265
    },
    {
      "item_id": 21018793542,
      "score": 0.795533015
    },
    {
      "item_id": 7965085370,
      "score": 0.7947649649999999
    },
    {
      "item_id": 21853369810,
      "score": 0.792521715
    },
    {
      "item_id": 12351221890,
      "score": 0.79030961
    }
  ],
  "mtime": 1677226180,
  "platform": 2,
  "play_cnt": 734,
  "rank": 0,
  "region": 1,
  "scenario": "INS_reels",
  "scores": "{\"third_party_pdp\":0.0000013471473,\"video_ppt_detect\":0.000100550096}",
  "state": 0,
  "task_id": "688",
  "vid": "CfWXG0jjcn3",
  "width": 720
}
```

其中，该项目 scenario 指定为 `INS_reels`。

## 输出

```json
{
  "aid": "CoR8abxDCnE",
  "author_id": "51223614878",
  "bit_rate": 1580908,
  "caption": "#fixgurlssid \n🎥 :\n•\n•\n•\n#hijaberstyle #hijabtrend #inspirasioutfit #inspirasioutfithijab #selebgramhits #hijabindo #selebgramhijab #hijabfashion #hijabers_indonesia #hijabtraveling #hijabstyle #outfitoftheday #ootdfashion #tutorialhijabmodern #hijabersindo #outfithijab #trendhijabootd #cantikberhijab #tutorialhijabpasmina #ootdhijabstyle #hijabtraveller #ootdhijabkece #tutorialhijab #tutorialhijabplisket #hijabfashion #ootdhijabindo #ootdhijab",
  "cover_image_mms_url": "http://img.sp.mms.shopee.sg/sg-11134113-23020-5x03xh081vnv5b",
  "ctime": 1677152282,
  "detect_lang": "en",
  "duration": 10233,
  "hashtag": "hijabers_indonesia",
  "height": 1024,
  "id": 729,
  "job_id": "20230223185622llfHzT",
  "keyword": "",
  "l1_topic": "Talent",
  "l2_topic": "Dance",
  "last_modify_time": 1675599446,
  "mms_url": "https://vod.sp-cdn.shopee.com/c3/98934353/122/A3oyOXd3AMRgQ=gZEeAKACY.mp4",
  "mms_vid": "sg_3e3bce99-eb4e-4479-937d-60a74df3a6d7_000345",
  "mmu_item_image_list": [
    {
      "idx": 4,
      "score": 1,
      "url": "http://proxy.uss.sz.shopee.io/api/v4/11110901/mms/sg-11110901-23020-xxx4t5881vnv09_sprite1x1-21"
    }
  ],
  "mmu_item_list": [
    {
      "item_id": 6517544075,
      "score": 0.75624737
    },
    {
      "item_id": 9039903329,
      "score": 0.75413163
    },
    {
      "item_id": 9080877501,
      "score": 0.7421407
    },
    {
      "item_id": 17768856751,
      "score": 0.7413784999999999
    },
    {
      "item_id": 16538123399,
      "score": 0.74012938
    },
    {
      "item_id": 16422974644,
      "score": 0.7398663999999999
    },
    {
      "item_id": 10024920773,
      "score": 0.7391202
    },
    {
      "item_id": 2983187005,
      "score": 0.7363121
    },
    {
      "item_id": 10282792843,
      "score": 0.736138285
    },
    {
      "item_id": 12261397930,
      "score": 0.73560245
    }
  ],
  "mtime": 1677152278,
  "platform": 2,
  "play_cnt": 5868,
  "rank": 0,
  "region": 1,
  "scenario": "INS_reels",
  "scores": "{\"third_party_pdp\":0.000010948371,\"video_ppt_detect\":0.00033609892}",
  "state": 0,
  "task_id": "688",
  "vid": "CoR8abxDCnE",
  "width": 576,
  "item_list": [
    {
      "item_id": 18928606296,
      "shop_id": 48270519,
      "order_num_shopee_30d": 5279,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 18237304910,
      "shop_id": 214199461,
      "order_num_shopee_30d": 160,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 21649568526,
      "shop_id": 7263969,
      "order_num_shopee_30d": 147,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 19842866256,
      "shop_id": 9069060,
      "order_num_shopee_30d": 2710,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 20032218546,
      "shop_id": 381673651,
      "order_num_shopee_30d": 7,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 18950532754,
      "shop_id": 10221730,
      "order_num_shopee_30d": 6058,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 21074366900,
      "shop_id": 308748430,
      "order_num_shopee_30d": 3,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 14793926870,
      "shop_id": 22403521,
      "order_num_shopee_30d": 16,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 19157822939,
      "shop_id": 15311055,
      "order_num_shopee_30d": 196,
      "level1_global_be_category": "Women Clothes"
    },
    {
      "item_id": 20549875870,
      "shop_id": 185754854,
      "order_num_shopee_30d": 171,
      "level1_global_be_category": "Women Clothes"
    }
  ],
  "data_flag": "INS_reels"
}
```

所有输入端 kafka 字段全部透传到输出端 kafka，另外加入：

1. `item_list`：视频匹配到的商品结果，属性包括：item_id、shop_id、order_num_shopee_30d、level1_global_be_category

2. `data_flag`：每天跑的流程该字段不变，内容与scenario字段相同，均为INS_reels。

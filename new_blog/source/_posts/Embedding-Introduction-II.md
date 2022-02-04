---
title: Embedding Introduction (II)
date: 2022-01-20 01:21:10
tags: 
- Embedding
- Item2Vec
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction-II/item2vec_top.png
top_img: /img/Embedding-Introduction-II/item2vec_top.png
---

#### 生成 Embedding 的方法 (II)

#### Item2Vec : Word2Vec 的衍生

Word2Vec 那种用向量 embedding 去表达单词的方法因其 state-of-art 的性能而在 NLP 领域备受关注。同样的，这种技术/技巧在 推荐系统领域也受到了追捧。特别的，通过使用 skip-gram using negative sampling 的 i2i 协同过滤(CF) 算法因为比肩 SVN CF 算法而受到推崇。这边博客，也主要讲解 Item2Vec 中的一些特点。

#### Skip-gram with negative sampling (SGNS)

Reference: [Efficient estimation of word representations in vector space](https://arxiv.org/pdf/1301.3781.pdf%C3%AC%E2%80%94%20%C3%AC%E2%80%9E%C5%93)

SGNS 方法旨在找到单词的表达，使其能够抓住该单词与句子中周围单词的关系。

我们用 $(w_i)^K_{i = 1}$ 表示一批连续的单词，其中，这些词都来自于一个有限词汇集 $W = \{w_i\}^W_{i=1}$。 从而，我们可以表达 skip - gram 的目标，最大化：

$$
\frac{1}{K} \sum^K_{i = 1} \sum_{-c <= j <= c, j \neq 0} \log{p(w_{i + j} | w_i)} \tag{1}
$$

其中，$c$ 是上下文的窗口大小，该大小可能取决于 $w_i$。而 $p(w_{i + j} | w_i)$ 则是 softmax 函数：

$$
p(w_{i + j} | w_i) = \frac{\exp(u^T_iv_j)}{\sum_{k\in I_w}\exp(u^T_iv^T_k)}
$$

其中 $u_i\in U(\mathcal{R^m}) $ 和 $v_i \in V(\mathcal{R^m})$ 分别是 $w_i \in W$ 中关于目标和上下文表达的隐向量。$m$ 通常通过经验与数据集的大小去选择。使用 softmax 函数不太现实，因为计算 $\nabla p(w_{i + j} | w_i)$ 的时间复杂度是线性的，与词汇集的大小有关。

##### 负采样

为了不大幅影响精度的情况下环节上述计算问题。我们将上面的 softmax 函数换成了 

$$
p(w_{i + j} | w_i) = \sigma(u^T_iv_j)\prod^N_{k=1} \sigma(u^T_iv^T_k)
$$

其中，$\sigma(x) = \frac{1}{1 + \exp(-x)}$，$N$ 是一个决定每个正样本所需负例样本数的参数。而 $w_i$ 中的负样本原则概率通过一元模型函数（Unigram Distribution） $f(w_i)$ 的 $\frac{3}{4}$ 次方来获得。整个公式为：

$$
P(w_i) = \frac{f(w_i)^{3/4}}{\sum_{j = 0}^n(f(w_i)^{3/4})}
$$

这个一元函数实际上就是将样本中的所有词写到了一个数组中，重复的词重复写到数组的不同位置(使用每个单词的索引填充)，直接通过生成随机数的方式去返回该随机数对应位置的数。如此依赖，如果数组中出现次数多的数，被负采样的概率就更大。

使用 $\frac{3}{4}$ 次方，更多的是一种经验效果，好于单纯用一元函数模型。

##### 降采样

除此以外，为了解决稀有与高频词的不平衡问题，文章中提出了一种降采样的方法。

已知一个输入词序列，我们以一个概率去丢弃每个词：

$$
p(discard | w) = 1 - \sqrt{\frac{\rho}{f(w)}}
$$

其中 $f(w)$ 为该词的词频，$\rho$ 为一个预设阈值。这个操作据报道能提升训练进程，并大幅提升稀有词的表达。

以上就是**SGNS** 的两大操作 负采样 与 降采样。

#### Item2Vec 思路

SGNS 方法被使用到了 item-based 的协同过滤推荐系统中。有些场景，我们没办法达到用户与一系列商品之间的关联信息，这时候，通过商品侧做的协同过滤就尤为重要。使用 SGNS 的想法也很显然，只要我们把一集合的商品认为是一序列的单词，则使用 embedding 去获得商品间的相关性与获得单词之间的相关性就并无不同。

从序列到集合后，时空信息就丢失了，也就是丢掉了它们之间的相邻关系。而 Item2Vec 就考虑丢掉这部分信息，他们认为在同一个 set 中的商品就应该是相关的，无论用户看到这个 set 的顺序或者时间是如何的。虽然这个假设在其他场景下不成立，不过可以认为在当前场景下是合理的。

因此，我们认为每对在同一个子集合中的商品是正例。这意味着集合的大小决定了窗口的大小。特别地，对于一个给定集合的商品，方程 $(1)$ 变为 

$$
\frac{1}{K} \sum^K_{i = 1} \sum^K_{j \neq i} \log p(w_j | w_i)
$$

整个集合大小为 $K$，计算两两 pair 之间的 log 概率之和。

另一种方法，则保持方程 $(1)$ 不变，在执行期间 shuffle 每个集合内的商品。实验中发现两种方法最终的表现差不多。

后面的步骤与上一个 Section 保持一致，这就是 Item2Vec。工作中，我们使用 $u_i$ 作为第 $i$ 个商品的表达，而一对商品的相关性通过 cosine 相关性计算得到。另一种方法是使用 $v_i$，或者 $v_i + u_i$，或者它俩的 concatenation $[u^T_iv^T_i]^T$。最后两种方法优势有更好的效果。

#### Item2Vec 效果

将 Item2Vec 生成的 embedding 用作聚类，与基于 SVD 方法的用户 embedding 聚类结果进行比较，向量维度保持一致，即 $m = 40$。对比数据为音乐领域里 web 音乐人根据类别的聚类，同一个颜色的节点表示相同类型的音乐人。对比结果如下

![item2vec.jpg](https://jason24-zeng.github.io/img/Embedding-Introduction-II/item2vec.jpg)

可以看出，两者差距不大，甚至 Item2Vec 的效果更好。

#### Reference

[Item2Vec: Neural Item Embedding for Collaborative Filtering](https://arxiv.org/pdf/1603.04259v2.pdf)

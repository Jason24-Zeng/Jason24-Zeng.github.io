---
title: Embedding Introduction (III)
date: 2022-01-20 22:23:59
tags: 
- Embedding
- Graph Embedding
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction-III/deepwalk_top.png
top_img: /img/Embedding-Introduction-III/deepwalk_top.png
---

### Embedding 在互联网场景的使用 - Graph Embedding

之前讲的 word2vec 和 item2vec 实际上都是在一个序列的基础上获得对象的隐式向量表达的。但是在互联网场景下，数据对象之前可能更多得呈现的是图结构，比如使用用户行为生成的物品全局关系图，或者物品属性知识图谱等。这种背景下，传统的序列 embedding 方法无法很好处理，因此，对图结构中节点进行表达的 graph embedding 便成了新的研究方向。随着大数据时代的到来，推荐系统愈发重要，这项技术也因此愈发受到重视。

### Graph Embedding 早期方法 - Deep Walk

基于随机游走生成图结构对应的节点序列，是早期 Graph Embedding 的一种重要方法。其允许重复访问已访问节点，使用深度优先（DFS）进行序列生成。

其思路是，给定一个节点，通过随机采样的方法，获得该节点相邻节点中的一个，以此作为序列的下一个节点。重复上述步骤，知道序列长度达到预期长度。再使用 skip-gram word2vec 的方法表达向量。使用了 Hierachical Softmax 中构造 Huffman 树的技巧优化了 softmax 的时间复杂度

其伪代码表示为：

![graph embedding deep walk algorithm](https://jason24-zeng.github.io/img/Embedding-Introduction-III/fake_coding.png) 

使用随机游走的方法创建训练数据，这种方法的特点：

- 可以实现机器、进程或线程维度的并行

- 适合动态更新，更新时只对新增节点增加

该算法的超参数有：

- skip-gram 的窗口大小 $w$

- 生成向量的维度，隐藏层的神经元个数 $d$

- 每个节点的游走次数 $\gamma$

- 游走长度 $t$

可认为 DeepWalk 是 Graph Embedding 的 baseline 方法。

参考：[DeepWalk: Online Learning of Social Representations](https://arxiv.org/pdf/1403.6652.pdf)

### Large-scale Information Network Embedding (LINE)

LINE 将非常大的信息网络映射到了低维向量空间（embedding）中，对可视化，节点分类等问题都有不错的效果。其相对 DeepWalk 这种纯粹随机游走的序列生成方式有以下两个特点：

1. 可以应用到有向图、无向图以及边有权重的网络

2. 使用一阶、二阶的临近关系引入目标函数，从而使最终学出的 node embedding 的分布更加均匀平滑，避免 node embedding 聚集的情况发生

3. 适用于大规模 network 上进行应用(Deep Walk 也适用)

#### 问题定义

##### Information Network

$G = (V, E)$ 其中 $V$ 是 vertex 节点集合，$E$ 是 edge 边集合。定义 $e = (u, v) \in E$ 是一个有序的节点对，边权 $W_{uv} > 0$。如果 $G$ 为无向图 (undirected graph)，则有 $(u, v) = (v, u)$ 与 $W_{uv} =  W_{vu}$。而如果 $G$ 为有向图 (directed graph)，那么有 $(u, v) \neq (v, u)$ 与 $W_{uv} \neq W_{vu}$

##### First-order Proximity (local network structure) 一阶近邻关系

边权 $W_{uv}$ 就表示节点 $u$ 与 $v$ 之间的 first-order proximity。两个节点之间相连的边权重越高，则这两个点越相似。若两点之间没有边，则 first-order proximity 为 0

##### Second-order Proximity (global netword structure) 二阶近邻关系

首先定义一个 $P_u = (W_{u,1},..., W_{u, |V|})$ 便是节点 $u$ 与其他所有节点的 first-order proximity 组成的集合。$u$ 与 $v$ 之间的相似度，则可以通过 $P_u$ 与 $P_v$ 之间的权重重合程度来决定相似度。两个节点的 neighbors 重复得越多，两者越相似。特别的，如果没有节点同时指向 $u$ 与 $v$ 或者 被 $u$ 与 $v$ 所指向，则 $u$ 与 $v$ 之间的 second-order proximity 为 0。

如下图的一个 Information Netword 的 toy model，可以看出 5 与 6 之间有较高的 second-order proximity，而 6 与 7 之间有较高的 first-order proximity。

![Information Network](https://jason24-zeng.github.io/img/Embedding-Introduction-III/line_information_network.png)

##### Large-scale Information Network Embedding

使用一个低维向量去表示整个 Information Network，即学习一个函数：

$$
f_G : V \rightarrow R^d, where \ d \ll|V|
$$

#### 模型描述

如前面所说，一个好的 Information Network Embedding 既需要有 first-order proximity 的信息，也不能忽视 second-order proximity。

##### 考虑 first-order proximity 进行建模

对于每个无向边 $(i,j)$，我们可以定义点 $v_i$ 与 $v_j$ 的联合概率如下：

$$
p_1(v_i, v_j) = \frac{1}{1 + \exp(-\vec{u_i}^T\cdot\vec{u_j})} \tag{1}
$$

其中，$\vec{u_i}\in R^d$ 是节点 $v_i$ 的低维向量表达。方程 $1$ 定义了一个在空间 $V \times V$ 的分布函数，它的经验概率可以被定义为 

$$
\hat{p_1}(i,j) = w_{ij}/W
$$

其中，

$$
W = {\sum_{(i, j)\in E} w_{ij}}
$$

为了维护 first-order proximity，一个直白的想法是最小化目标函数 

$$
O_1 = d(\hat{p}_1(\cdot ,\cdot), p_1(\cdot ,\cdot)) \tag{2}
$$

其中 $d(\cdot ,\cdot)$ 为两个分布之间的距离，这个距离我们选择使用 KL 散度来表达，同时取出其中的常数项，我们就有：

$$
O_1 = - \sum_{(i, j)\in E} w_{ij}\log p_1(v_i, v_j) \tag{3}
$$

需要注意到 first-order proximity 只对无向图有用。通过找到集合 $\{\vec{u_i}\}_{i = 1..|V|}$ ，使方程 $3$目标函数最小化，我们可以在 $d$ 维空间中表示每个节点。

##### 考虑 second-order proximity 建模

second-order proximity 对有向图与无向图都可行。在考虑 second-order proximity  是，节点扮演了两个角色：

- 节点本身

- 其他节点的上下文 context

为了区分这两个角色，我们引入了两个向量 $\vec{u}_i$ 与 $\vec{u}'_i$，分别对应魔偶个节点 $v_i$ 的上述两种角色的表达。

首先我们定义一个上下文 $v_j$ 被 $v_i$ 产生的概率:

$$
p_2(v_j|v_i) = \frac{\exp(\vec{u}_i'^T \cdot \vec{u}_j)}{W(\vec{u}_j)} \tag{4}
$$

其中，

$$
W(\vec{u}_j) = \sum \exp(\vec{u}_k'^T \cdot \vec{u}_j)
$$

该方程实际定义了一个网络里所有节点为节点 $v_i$ context 的条件概率。如果 $p2(\cdot | v2)$ 与 $p2(\cdot | v1)$ 的概率分布相似，则这两个点的 second-order proximity 是相似的。

为了维护 second-order proximity，我们需要逼近经验分布函数 $\hat{p}_2(\cdot | v_i)$，因此，我们需要最小化一下目标函数

$$
O_2 = \sum_{i \in V} \lambda_i d(\hat{p}_2(\cdot|v_i), p_2(\cdot|v_i)) \tag{5}
$$

因为网络中每个节点的重要性不同，我们引入 $\lambda_i$ 表达网络中节点 $i$ 的优先级，它可以通过度数衡量 (需要进一步探讨，应该是图论中的出入度计算) 或者通过一些算法预估 (比如 PageRank)。

经验分布 

$$
\hat{p_2} (\cdot \vert v_i) = w_{ij} / d_i
$$

其中 $w_{ij}$ 为边 $(i,j)$ 的权重， $d_i = \sum_{k\in N(i)}  w_{ik}$ 表示顶点 $i$ 的出度 (out-degree), $N(i)$ 则是节点 $v_i$ 的 out-neighbors。

在文章中，简单得将 $\lambda_i$ 设成顶点 $i$ 的度数，即 $\lambda_i = d_i$， 同时采用 KL 散度作为距离方程，同时去掉常数项，则最终我们目标函数变为 

$$
O_2 = - \sum_{(i, j)\in E} w_{ij} \log p_2(v_j|v_i)
$$

通过学习集合 $\{\vec{u_i}\}_{i = 1..|V|} $ 与 

$ \{\vec{u_i}'\}_{i = 1..|V|}$ 最小化目标，我们可以用一个 $d$ 维向量 $\vec{u_i}$ 表达每一个顶点。

##### Combine 一阶与二阶 proximity

文章中分别训练一阶 proximity 和 二阶 proximity，然后将得到的 embedding 拼起来作为该顶点的 embedding 表达。一个更原则性的方式是将目标函数联立起来求。

参考：[LINE: Large-scale Information Network Embedding](https://arxiv.org/pdf/1503.03578.pdf)

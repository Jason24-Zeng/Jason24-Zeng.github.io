---
title: Embedding-Introduction (IV)
date: 2022-01-21 18:08:34
tags: 
- Embedding
- Graph Embedding
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction-III/deepwalk_top.png
top_img: /img/Embedding-Introduction-III/deepwalk_top.png
---

#### LINE

接 [上一章节](https://jason24-zeng.github.io/2022/01/20/Embedding-Introduction-III/)

我们已经定义好处理 LINE 的一阶与二阶 proximity 的目标函数，这一节主要是讨论函数的优化。

#### 模型优化

###### 回顾

first-order proximity 目标函数

$$
O_1 = - \sum_{(i, j)\in E} w_{ij}\log p_1(v_i, v_j) \tag{1}
$$

second-order proximity 目标函数

$$
O_2 = - \sum_{(i, j)\in E} w_{ij} \log p_2(v_j|v_i) \tag{2}
$$

计算 second-order proximity 消耗较高，因为需要计算每个顶点的条件概率，然后把这些概率加和。为了解决这个消耗问题，文中采用了负采样的方法，通过每条边的一些噪音分布去采样多条负边。对边 $(i, j)$ 的目标函数变为：

$$
\log\sigma(\vec{u_j}^{1T} \cdot \vec{u_i}) + \sum^K_{i = 1} E_{v_n \sim P_n(v)}\left[\log\sigma(\vec{u}_n^{1T} \cdot \vec{u}_i)\right] \tag{3}
$$

其中，$\sigma(x) = 1 / (1 + \exp(-x))$ 是 sigmod 函数。前一项基于观察的边建模，后一项对从噪音分布采样得到的负边建模。$K$ 表示负边的总数。我们设定 $P_n(v) \propto d_v^{3/4}$，而 $d_v$ 为顶点 $v$ 的出度。

对于目标函数 $1$， 存在一个平凡解：$\forall i k, u_{ik} = \infty$。为了避免这个问题，我们依然使用方程 $3$ 中的负采样方法，只是把 $\vec{u_j}^{1T}$ 换成 $\vec{u_j}^{T}$ 即可。

为了最优化方程 $3$，文中采用了异步随机梯度算法（asynchronous stochastic gradient algorithm, ASGD）。每一步，ASGD 算法采样了一 mini-batch 的边，然后更新模型参数。如果一条边 $(i, j)$ 被采样，则梯度 (顶点 $i$ 的embedding 向量 $\vec{u_i}$) 会被使用一下方程计算

$$
\frac{\partial O_2}{\partial \vec{u_i}} = w_{ij}\cdot \frac{\partial \log p_2(v_j|v_i)}{\partial \vec{u_i}}
$$

这样的问题在于梯度会被乘以边权，当边权方差较大时，可能出现问题。

上面的问题主要出在对于不同的边权，固定的学习率不合适。所以一个简单的解决方法是将边权为 $w$ 的边 unfold 成 $w$ 条 0 - 1 边。这可以解决问题，但是会显著提升内存占用。文中的解决方法实际上是，将 $w$ 作为作为概率分布的权重，随机采样样本边，将取到的样本边作为 0 - 1 边去训练。这个想法与 XGBoost 中的直方图思想如出一辙。

而随机采样要达到空间时间上的最优还要考虑一些算法上的优化。我们假设 $W = (w_1, w_2, ..., w_{|E|})$ 表示一系列边的权重，我们可以通过计算一个 $w_{sum} = \sum_{i = 1}^{|E|}$ 作为随机数的上界，判断这个随机数处于哪个区间，便能 sample 出边，这样的算法时间复杂度是 $O(|E|)$ ，在边很多的情况下时间花销较大。为了解决这个问题，可以考虑使用 [alias table 方法](https://blog.csdn.net/haolexiao/article/details/65157026)，这个方法可以再 $O(1)$ 的时间复杂度下从固定的离散分布里根据权重筛选样本。

由此，我们可知，采样一条边需要 $O(1)$ 的时间复杂度，那么负采样优化后整体的时间花销为 $O(d(K + 1))$ ，$K$ 为负采样数量。因此，每一步迭代会花费 $O(dK)$ 次。而实际情况下，每次消耗的时间正比于边的数量 $|E|$。因此，LINE 的整体时间复杂度是 $O(dK|E|)$，与顶点数量无关。

###### 总结

从上面的理论推到可以看出，边的采样策略在不影响整体效果的前提下提升了随机梯度下降的效率。

# 

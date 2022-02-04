---
title: Embedding Introduction (VI)
date: 2022-01-22 09:01:10
tags:
- Embedding
- SDNE
- Graph Embedding
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction-VI/SDNE_top.png
top_img: /img/Embedding-Introduction-VI/SDNE_top.png
---

#### SDNE : Structural Deep Network Embedding

DeepWalk, LINE, Node2vec 这几个 Graph Embedding  的方法，在训练 Embedding 的时候使用的都是 skip-gram 的模型结构。这个模型结构只有一层隐藏层，很难学到一些高阶非线性的特征。而 SDNE 为了表达高阶非线性，引入了更深的网络结构。

#### 训练网络结构

SDNE 使用的训练框架是一种半监督式的深度模型训练框架。其整体的网络结构如下：

![SDNE_framework](https://jason24-zeng.github.io/img/Embedding-Introduction-VI/SDNE_top.png)

首先搞清楚一边的深层网络结构 ： Autoencoder

##### Autoencoder for second-order proximity

![SDNE_autoencoder](https://jason24-zeng.github.io/img/Embedding-Introduction-VI/autoencoder.png)

如上图，为一个 autoencoder 的基本结构。我们自下往上的看。

输入层 $x_i \in R^{|V|}$ ， $|V|$ 为所有节点的数量。如果 $(i, j)$ 边存在，则 $x_{ij} > 0$，可以理解为 $x_{ij} = w_{ij}$ 为 edge $(i,j)$ 的边权，否则，$x_{ij} = 0$。

紫色部分是 encoder，是将稀疏向量 $x_i$ 降维，使之稠密的过程。

中间蓝色部分就是每个顶点 $i$ 对应的输出 embedding $y_i^{(K)}$ 。

深蓝色部分则是 decoder，可以认为是紫色部分的 revert，升维，使之稀疏，最终输出向量 $\hat{x_i}$。

整个 autoencoder 的过程是无监督的。在迭代过程中，我们需要计算输入向量 $x_i$ 与向量 $\hat{x_i}$ 的 loss，认为是二阶近邻关系的目标函数表达。

这个 loss 表示为：

$$
\begin{aligned}
\mathcal{L}_{2nd} &=  \sum^n_i \|(\mathbf{\hat{x}}_i - \mathbf{x}_i) \bigodot  \mathbf{b}_i\|^2_2  \\\\
&= \|(\hat{X}_i - X_i) \bigodot  B \|^2_F
\end{aligned}
$$

其中 $\bigodot$ 表达 Hadamard 乘积，即按位置相乘之和。如果 $s_{i,j} = 0$，则 $b_{i,j} = 1$。否则 $b_{i,j} = \beta > 1$。因此，通过将临接矩阵 $S$作为输入，使用修改的深层 autoencoder，有相似邻居结构的顶点会在表达空间 embedding 中靠近。

##### Supervised component for first-order proximity

除了 global 的网络结构，我们也需要抓住局部结构，这里的局部结构在文中专指 first-order proximity。损失函数可从网络图中得到：

$$
\begin{aligned}
\mathcal{L}\_{1st} &= \sum^n\_{i,j = 1} s\_{i,j}\|y^{K}\_i - y^{K}\_j\|^2\_2\\\\
&= \sum^n\_{i,j = 1} s_{i,j}\|y\_i - y\_j\|^2\_2
\end{aligned}
$$

(这里出现 markdown 与 mathjax 对 _ 的转义出现冲突，导致数学公式显示有问题。需要再 _ 前加下划线解决。)

这个损失函数表征的是，如果两个节点相连，则存在一定的相似性，如果权重越大，相似性越高。公式中的 $y$ 表示自编码器的中间层输出。

##### 整个模型损失函数

整个模型的损失函数结合了一阶和二阶 proximity 相关的目标函数，同时引入了一个正则项：

$$
\mathcal{L}\_{reg} = \frac{1}{2}\sum^K_{k = 1}(\|W^{(k)} \|^2_F + \|\hat{W}^{(k)} \|^2_F)
$$

其中 $W^{(k)}$ 与 $\hat{W}^{(k)}$ 分别代表自编码器中 encoder 和 decoder 网络的权重。

整体损失函数如下：

$$
\begin{aligned}
\mathcal{L}\_{mix} &= \mathcal{L}\_{2nd} + \alpha\mathcal{L}\_{1st} + \mu\mathcal{L}\_{reg} \\\\
&= \|(\hat{X}\_i - X\_i) \bigodot  B \|^2\_F + \alpha \sum^n\_{i,j = 1} s\_{i,j}\|y\_i - y\_j\|^2_2 + \mu\mathcal{L}\_{reg}
\end{aligned}
$$

这个损失函数在实验效果上明显由于 LINE，但是因为网络结构更加复杂，输入输出中又分厂稀疏，因此时间复杂度较高，是一个优化的方向。

#### Reference

[Structural Deep Network Embedding](https://www.kdd.org/kdd2016/papers/files/rfp0191-wangAemb.pdf)

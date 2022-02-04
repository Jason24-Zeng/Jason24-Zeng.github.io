---
title: Embedding Introduction (V)
date: 2022-01-21 19:49:07
tags:
- Embedding
- Node2Vec
- Graph Embedding
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction-V/node2vec_top.png
top_img: /img/Embedding-Introduction-V/node2vec_top.png
---

### DeepWalk 的优化 - Node2Vec

可以这么理解，deepwalk 主要是基于 DFS 邻域的 graph embedding 算法，而 LINE 则是基于 BFS 邻域的 graph embedding 算法。接下来介绍的 node2vec 则是总和考虑 DFS 与 BFS 的 graph embedding 算法，是结合 DFS 与 BFS 的随机游走的 deep walk。

#### 算法原理

定义：

1. $f(u)$ 为顶点 $u$ 到其对应 embedding 向量的映射

2. $N_S(u)$ 为在采样策略 $S$ 下顶点 $u$ 的近邻顶点集合

而 node2vec 优化的目标比较直接，就是希望找到最优映射 $f(u)$，使近邻顶点出现的概率最大化，即

$$
max_f\sum_{u \in V} \log Pr(N_s(u) | f(u))
$$

为了使该最优化问题可解，文章做了两个假设：

1. 条件独立性假设。即在给定顶点 $u$ 下，其近邻顶点出现的概率，与近邻集合中其他顶点无关。即 
   
   $$
   Pr(N_s(u) | f(u)) = \prod_{n_i\in N_s(u)} Pr(n_i | f(u))
   $$

2. 特征空间对称性假设。与 LINE 不同，假设一个顶点无论是作为原顶点还是近邻顶点，其共享同一套 embedding 向量。对应表达为
   
   $$
   Pr(n_i | f(u)) = \frac{\exp(f(n_i)\cdot f(u))}{\sum_{v\in V} \exp(f(v)\cdot f(u))}
   $$

最终，根据假设条件，我们的最终目标函数表示为

$$
max_f\sum_{v\in V}\left[-\log Z_u + \sum_{n_i\in N_s(u)} f(n_i) \cdot f(u)\right]
$$

其中 $Z_u = \sum_{n\in N_s(u)} \exp(f(n_i)\cdot f(u))$ 的计算代价高，需要采用负采样技术进行优化。

#### 采样策略

相比随机游走 deep walk 中的采样方式， node2vec 也是随机游走，但是是一种依概率随机游走，这种随机游走是有偏的。

比如给定当前顶点 $v$，访问下一个顶点 $x$ 的概率我们设定为 

$$
P(c_i = x | c_{ - 1} = v) = \frac{\pi_{vx}}{Z} \text{, if } (v, x) \in E
$$

$$
P(c_i = x | c_{i - 1} = v) = 0  \text{, otherwise}
$$

其中 $\pi_{vx}$ 表示顶点 $v$  与顶点 $x$ 之间的转移概率（未 normalized）， $Z$ 则是归一化常数

文中 node2vec 引入两个参数 $p$ 与 $q$ 来控制随机游走的策略。

$$
\pi_{vx} = \alpha_{pq}(t, x)\cdot w_{vx}
$$

其中 $w_{vx}$ 表示顶点 $v$ 与 $x$ 之间的边权。而 $\alpha_{pq}$ 则与上一次游走起点 $t$ 和当次游走终点 $x$ 有关，可知两个顶点的距离 $d_{tx}$ 可以有三个取值：

- $d_{tx} = 0$，即两点属于同一个点，此时设定 $\alpha_{pq}(t, x) = \frac{1}{p}$，所以 $p$ 又被称为 return parameter，表示马上重新访问原来 node 的可能性。
- $d_{tx} = 1$，即 $t, x$ 与 $v$ 三点均相邻，此时设定 $\alpha_{pq}(t, x) = 1$
- $d_{tx} = 2$，即 $x$ 远离 $t$ 点，两点不直接相邻，此时设定 $\alpha_{pq}(t, x) = \frac{1}{q}$，所以 $q$ 又被称为 in-out parameter。

从解释中可以看出， $p, q$ 值的设定，描述了算法对 BFS 与 DFS random walk 的倾向性。具体的取值可以通过下图直观看出:

![node2vec_parameter](https://jason24-zeng.github.io/img/Embedding-Introduction-V/node2vec_pm.png)

#### 算法策略

采样顶点序列后，后续的方式与 deepwalk 与 LINE 一致，均通过 word2vec 的方法去学习 embedding 向量。node2vec 与 LINE 一样，是依概率抽取邻接点，同时采用 alias table 算法进行采样。

其核心算法伪代码如下：

![node2vec_algorithm](https://jason24-zeng.github.io/img/Embedding-Introduction-V/node2vec_algorithm.png)

#### Reference

[node2vec: Scalable Feature Learning for Networks](https://www.kdd.org/kdd2016/papers/files/rfp0218-groverA.pdf)

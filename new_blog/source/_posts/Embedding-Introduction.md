---
title: Embedding Introduction (I)
date: 2022-01-19 21:26:50
tags: 
- Embedding
- Word2Vec
categories:
- Machine Learning
- Embedding
cover: /img/Embedding-Introduction/embedding_top.jpeg
top_img: /img/Embedding-Introduction/embedding_top.jpeg
---

#### 什么是 Embedding ?

广义得说， Embedding 就是一种用向量表达某个词汇的方式。其特点就是可以捕捉到文档中单词的上下文、语义与句法相似以及与其他单词的关系等。

考虑一下我们之前在做类别特征时常用的 one-hot 方法，假设整个上下文的单词量有 L，对任意一个单词，我们将其做 one-hot 转换成一个长度为 L 的向量，其中某个位置为 1，其余位置为 0。这样我们的确可以将单词映射到一个向量表达上。但当前还有一个问题：这样的表达可以表征两个单词的相关性么？

答案是不能，因为两个单词的计算 cosine 为 0。(在不同位置包含一个 1)

所以怎么才能体现相关性呢？可以考虑将这些 one-hot 矩阵映射到一个低位空间，使其向量值变稠密，同时保证“相似”词的 embedding 内积接近 1，不相似词的 embedding 内积接近 0，“相反”词的 embedding 内积接近 -1。这样的向量表达，就是我们常说的 embedding。

但是，这样的 embedding 需要如何生成呢？这篇文章就主要讲讲生成 embedding 的几种方法。

#### 生成 word embeddings 的方式 (一)

##### Word2Vec

word2vec 是一种使用浅神经网络训练 word embedding 的一种流行技巧。它主要通过两种包含神经网络的方式获得 embedding:   Skip Gram 和 CBOW (Common Bag Of Words)

而 embedding 从模型的什么部分获得呢？当模型训练完后，实际上我们得到了神经网络的权重，因为输入层是 one-hot 格式，到隐藏层时，相当于只有这个词对应的 embedding 被激活了，因而可以用这个 embedding 来表达该单词。

###### CBOW 模型

CBOW 模型：使用每个词的上下文作为输入，以预测准确这个词为训练目标，去训练整个神经网络模型。

只包含一个上下文单词 one-hot 作为输入的神经网络整体实际结构如下：

![CBOW_word2vec.png](https://jason24-zeng.github.io/img/Embedding-Introduction/CBOW_word2vec.png)

这个输入，或者说这个上下文单词是一个大小为 V 的 one-hot 编码向量。隐藏层包含 N 个神经元，而输出则同样是一个 Size 为 V 的向量，其中的值通过了 Softmax 做归一化。目的是预测目标单词。

我们可以看到其中有两个权重矩阵 $W_{vn}$ 和 $W'_{nv}$ 。

这个神经网络中，唯一牵涉到非线性的地方就是输出层的 softmax，而没有用到其他激活函数，比如 sigmoid, tanh 或者 ReLU。

同样的，如果我们考虑的上下文单词不是一个，而是多个，则可以考虑如下结构：

![CBOW_word2vec_2.png](https://jason24-zeng.github.io/img/Embedding-Introduction/CBOW_word2vec_2.png)

将 C 个上下文单词考虑入模型，均使用权重矩阵 $W_{vn}$ 计算隐藏层输入，再对所有 C 取平均值得到输出层的输入。这样，我们就可以使用上下文单词产生词表达。

###### Skip-Gram 模型

整体结构如下，从某种意义上，这个结构就像 CBOW 结构翻转过来一样

![skip_gram.png](https://jason24-zeng.github.io/img/Embedding-Introduction/skip_gram.png)

对每个上下文位置，我们会得到 C 个 向量来预测词的可能性分布，每个向量对应一个单词。

两种模型都使用 backward propagation 负反馈的方式去迭代学习。

###### 优势场景

对于少量数据，skip gram 工作表现得更好，并且更能表达稀有词语。

CBOW 模型则对更频繁出现的词语表达更好，且更快。

###### 算法优化

文章中还提到了使用 Hierachical Softmax 和 Skip Gram 负采样的方式，使计算变得更有效率。详细文章可以参考[原文](https://arxiv.org/abs/1310.4546) 或[Xin Rong 的论文](https://arxiv.org/pdf/1411.2738.pdf)

##### 总结

本文主要介绍了一下词表达的一种形式 embedding，并简单地讲述了 embedding 的两种基础训练方法：CBOW 和 skip-gram。

在学习过程中，发现一个有趣的地方。通过输入层与隐藏层间权重矩阵的某一列来表达某个单词，这个做法和 Factorization Machine 中某个特征的表达所用的矩阵有异曲同工之妙。同时，也希望自己能在一点点积累知识的同时能继续提升自己的写作能力。 并能手动复现这个基础的训练代码。``

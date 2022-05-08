---
title: C++ Standard Library
date: 2022-05-08 17:04:55
tags:
-STL
-C++
category:
-C++
---

## 前言

作为一名进入职场快一年的员工，如果还没有去具体了解标准库的内容，以及其实现和构造，会是一件比较 shameful 的事情。为了弥补这块缺失，我特别开启这篇博客，用以记录在学习 C++ 标准库中的所知，以及可能的所感。



## Container 容器

容器的分类（按照是否为标准库容器等标准）：

- 标准 STL sequence 容器：比如 `vector`, `list`, `deque` 和 `string`

- 标准 STL associative 容器： 比如 `set`, `multiset`, `map` he `multimap`

- 非标准 STL sequence 容器：比如 `slist` 和 `rope`， `slist` 是单向链表，而 `rope` 可以理解为是 `heavy-duty` 的 `string`

- 非标准 STL associative 容器，比如 `hash_set`, `hash_multiset`,`hash_map` 和 `hash_multimap`。

- 还有一些标准 non-STL 容器，包括 `arrays`, `bitset`, `valarray`, `stack`, `queue`, 以及 `priority_queue`

需要注意：

- `vector` 可以作为标准 associative 容器的替代，有时甚至能比标准 associative 容器在时间空间上表现得更好。

- 有些情况下， `vector<char>` 替代 `string` 可能有不错的效果。

同时，我们还有另一种标准去对标准 STL 容器进行分类，即

- `contiguous-memory containers` 连续内存容器。它会将元素储存在一个或者多个 chunk 的内存中，同一个 chunk 可以有超过一个容器元素。它的一个特点是，如果一个新元素被插入或者一个存在的元素被清除，同一个内存 chuck 中的其他元素不得不移动，从而给新元素制造空间，或者填充被清除元素所占用的空间。

- `node-based container` 节点型容器。内存的每一个 chunk 都只有一个元素。插入或删除容器元素只会影响指向节点的指针，而不会影响节点本身，所以当某物被插入或删除时，元素值不必被移动。容器代表了链表，比如 `list` 和 `slist` 都是标准的关联性容器（通过平衡树实现）。Note，非标准的 hashed 容器使用不同的 node-based 实现。

以下的一些问题可以作为我们选择容器的判断标准

1. 

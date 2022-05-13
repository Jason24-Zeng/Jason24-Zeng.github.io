---
title: C++ Standard Library
date: 2022-05-08 17:04:55
tags:
- STL
- C++
categories:
- C++
---

## 前言

作为一名进入职场快一年的员工，如果还没有去具体了解标准库的内容，以及其实现和构造，会是一件比较 shameful 的事情。为了弥补这块缺失，我特别开启这篇博客，用以记录在学习 C++ 标准库中的所知，以及可能的所感。

## Container 容器

### Item 1. 小心选择容器

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

1. 我们是否需要插入新元素到容器的任意位置？如果需要，那么 associative 容器就不能满足要求，只能选用 sequence 容器
2. 我们关注容器内元素的顺序么？如果不关注，那么 hashed 容器变成了一个可行的选择，否则，我们得避免使用 hashed 容器
3. 我们使用的容器一定要是标准 C++ 的部分么？如果是，我们就不能使用 hashed 容器，slist 和 rope
4. 我们要求是哪种迭代器？如果必须是random access 迭代器，我们就只能选用 vector, deque 和 string，我们也可能想选用 rope。如果是 bidirectional 迭代器，我们一定要避免 slist 以及 hashed 容器的一种常见实现。
5. 当插入和清楚发生时，避免移动存在的容器变量是否重要？如果重要，我们需要远离 contiguous-memory 容器。
6. 容器中的数据是否需要 layout-compatible with C(兼容 C)？如果需要，则我们只能使用 vector
7. 查询是否加速极端考虑（？）？如果是的，我们可能想要使用 hashed 容器，排好序的 vector 以及标准关联性 associative 容器。
8. 是否介意潜在的容器使用引用计数 (reference counting)？如果介意，我们可能不能使用 string, 因为需要 string 的实现都是引用计数的。我们也需要避免 rope，因为 definitive rope 的实现也基于 reference counting。如果我们必须表示字符串，我们可能需要考虑使用 `vector<char>`
9. 对插入和删除我们是否需要 transactional semantics （一种语义，不破坏原有容器内部内容。）？或者说，我们需要保障回滚插入和删除的能力么？如果需要，我们将想要使用 node-based 容器。如果我们需要对多元插入使用 transactional semantics ，我们可能想要使用 list，因为 list 是唯一为多元插入提供 transactional semantics 的标准容器。transactional semantics 对致力于写异常安全代码的程序员尤其重要。
10. 我们是否需要最小化迭代器，指针和引用的不合理性，如果需要，我们可能想要使用 node-based 容器，因为在这些容器上插入和删除绝不会使迭代器，指针，引用等失效。通常，在 contiguous-memory 容器中插入或删除可能会使容器内所有的迭代器，指针以及引用失效。
11. 对拥有一个 random access 迭代的 sequence 容器，假设没有东西被删除且插入只发生在 container 的尾部，让其指向数据的指针和引用不失效是否是有用的。这个 case 非常特殊，如果发生上述情况，`deque` 是我们的理想选择。（这个问题不太理解，需要回来 double check）

### Item 2. 小心容器独立代码的迷惑

Beware the illusion of container-independent code

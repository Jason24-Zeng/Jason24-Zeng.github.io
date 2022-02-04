---
title: Alias Table Sampling
date: 2022-01-23 00:54:10
tags: 
- Algorithm
- Probability
categories:
- Algorithm
- Probability
cover: /img/alias-table-sampling/alias_table_top.png
top_img: /img/alias-table-sampling/alias_table_top.png
---

### 离散按概率随机抽样算法 - Alias method

因为最近学习图嵌入 graph embedding 的相关操作与算法，发现无论在使用 deepwalk, LINE, node2vec 还是 SDNE 等图嵌入方法时，随机游走选择下一个节点都会用到一种按概率采样的方法，也就是 Alias 方法。第一次学习到这个算法，是在转专业刷知乎是看到的，想来也是很奇妙，果然有些优秀的算法容易被人提起并记住。这种方法因为其 $O(1)$ 的时间复杂度，大幅加速了候选集的生成。接下来的章节将主要讲解一些相关的随机抽样算法，最后再讲到 Alias method。

#### 问题阐述

假设候选集为 $M$ 个事件，用 $1, ... ,m$ 对这些时间编号。这些事件互斥，发生的概率为 $p_i, i = 1, ..., m$，满足 $\sum^m_{i = 1} p_i = 1$。问，如何产生一个事件发生器，根据发生概率去产生事件呢？

#### Solution 1：preSum 方法

步骤：

- 通过数组 $p_i$ 依照前缀和方法生成 prefix Sum 数组 `presum`。亦将概率密度函数"积分"成概率分布函数

- 产生 0 - 1 的随机数，判断随机数处于哪个概率区间，返回对应的事件。

复杂度分析：

- 时间复杂度: 产生 `presum` 数组 $O(n)$，返回随机事件使用二分法 $(O(logn))$

- 空间复杂度：维护 `presum` 数组 $O(n)$

代码：

```cpp
class preSum_method {
  private:
    vector<float> presum;
    std::random_device rd;
    std::default_random_engine eng(rd());
    std::uniform_real_distribution<float> distr(FLOAT_MIN, FLOAT_MAX);
  public:
    preSum_method(vector<float>& prob) {
        presum.reserve(prob.size() + 1, 0.0f);
        for (int i = 0; i != prob.size(); i++) {
            presum[i + 1] = presum[i] + prob[i];    
        }
    }

    int generate_case() {
        int left = 0, right = presum.size() - 2;
        float random_num = distri(eng);
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (random_num < nums[mid + 1] && random_num >= nums[mid]) return mid;
            else if (random_num >= nums[mid + 1]) left = mid + 1;
            else right = mid;
        }
    }
};
```

#### Solution 2 : 预设候选集法

步骤，

- 根据概率生成一个很大的候选集

- 通过随机数的值返回对应候选集位置的事件

复杂度分析：

- 产出候选集的时间复杂度 $O(N)$，$N$ 与精度有关，返回随机事件 $O(1)$

- 空间复杂度 $O(N)$

代码：

```cpp
class CandidateSet {
  private:
    vector<int> candidate_set;
    int P;
  public:
    CandidateSet(vector<float>& prob, int precision) {
        P = 10 ** precision;
        for (int i = 0; i != prob.size(); i++) {
            vector<int> temp(i, (int) (i * P));
            std::copy(temp.begin(), temp.end(), std::back_insert(candidate_set));
        }
    }

    int generate_case() {
        int random_number = rand() % P;
        return candidate_set[random_number];
    }
};
```

这种方法主要的弊端在于需要申请很大一块连续空间来存储候选集。

#### Solution 3 : Alias Method

本文的重点。主要的创新点在于建表环节。我们考虑到等概率抽样的时间复杂度为 $O(1)$，而对二项分布的时间进行抽样的时间复杂度也是 $O(1)$。因此，整个时间的思路变成了如何把依概率抽样的时间转变为等概率抽样。

alias method 考虑维护一个 alias 表，里面有 $M$(事件个数) 个值得数组，对数组中的每个元素，是一个二项分布参数的三元组`prob, lower_event, higher_event`。通过等概率选取数组的元素，再依概率选择二元事件，我们就可以完成依概率对多元事件的抽样。

步骤：

- 制表。两张表：
  
  - 等概率表，大小为 $M$
  
  - 维护两个队列，small, large 分别存放小于 1 和 大于 1 的时间下标
  
  - 每次从 small，large 中各取一个，用 large 里的值填补 small 的，使 small 的整体概率等于 1，然后根据 large 剩余的整体概率，将元素重新放回 large 或 small 中。
  
  - 所有的概率都等于 1

- 采样

代码：

```cpp
class AliasMethod {
  private:
    vector<pair<int, int>> alias_method;
    int multiplier;
    std::random_device rd;
    std::default_random_engine eng(rd());
    std::uniform_real_distribution<float> distr(FLOAT_MIN, FLOAT_MAX);
  public:
    CandidateSet(vector<float>& prob, int precision) {
        std::queue<int> large, small;
        multiplier = prob.size();
        alias_method.reserve(multiplier);
        for (int i = 0; i != multiplier; i++) {
            int new_prob = prob[i] * multiplier;
            if (new_prob < 1) {
                small.push(i);
            } else if (new_prob > 1) {
                large.push(i)
            }
            alias_method[i] = new_prob == 1 ? {new_prob, i} : {new_prob, -1};
        }

        while (!large.empty()) {
            int large_pos = large.front();
            int small_pos = small.front();
            auto& l = alias_method[large_pos];
            large.pop();
            auto& s = alias_method[small_pos];
            small.pop();
            l.first -= s.first;
            s.second = large_pos;
            if (l.first < 1) small.push(large_pos);
            else if (l.first > 1) large.push(large_pos);
        }

    }

    int generate_case() {
        int random_case = rand() % multiplier;
        float random_num = distri(eng);
        auto& temp = alias_method[random_case];
        return temp.first <= random_num ? random_case : temp.second;
    }
};
```

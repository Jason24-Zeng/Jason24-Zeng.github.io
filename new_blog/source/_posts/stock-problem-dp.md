---
title: 股票问题系列--动态规划求解
date: 2022-01-16 20:20:34
tags: 
- Leetcode 
- DP
categories: 
- Algorithm
- Dynamic Program
cover: /img/stock-problem-dp/stock_problem.jpeg
top_img: /img/stock-problem-dp/stock_problem.jpeg
---

#### 前言

最近刷到多道 Best Time to Buy and Sell Stock 系列的问题，想要探究一下这类题目的通用动态规划解法。为此，特写了此文，找出这类问题的练习，供有面试需求的相关同学学习。

##### 相关题目

1. [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/#/description)
2. [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/#/description)
3. [123. Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/#/description)
4. [188. Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/#/description)
5. [309. Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/#/description)
6. [714. Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/description/)

##### 通用问题描述

给定一个数组 `vector<int>& prices` ，这个数组代表了一个序列化的股票价格，在某些限制条件下，求能通过这些天的股票买卖获得的最大收益。

首先，我们来记录一下可能的限制条件：

- 交易次数 `int k`

- 交易冷却时间 `int time`

- 交易费用 `int fee`

以及一些注明 (notation)：

- 股票价格所处日期`int i`，第一天为 `0`

我们可以看到，上述的这类问题，其实都存在很明显的状态转移关系。

- 当天有两个状态，持有股票与不持有股票

- 当天的状态，会与前面某天的状态有关。比如如果今天持有股票，则要么昨天也持有股票，要么需要再冷却期内不持有股票。

#### 通用问题状态转移关系

我们定义一个三维矩阵`T`， 其中 `T[i][k][0]` 表示累计到第 `i` 天，进行了最多 `k` 次交易，并在第 `i` 天手上不持有股票下的最大收益；同样的，`T[i][k][0]` 表示累计到第 `i` 天，进行了最多 `k` 次交易，并在第 `i` 天手上持有股票的最大收益。目前我们可以依此写出base 条件与递归条件：

1. base 情况：
   
   ```cpp
   // 开始计算股票前一天的初始化，不持有股票认为是合法的，为 0，持有股票认为是不可能
   T[-1][k][0] = 0, T[-1][k][1] = INT_MIN
   // 无可交易次数时的初始化，不持有股票认为是合法的，为 0，持有股票认为是不可能
   T[i][0][0] = 0, T[i][0][1] = INT_MIN
   ```

2. 递归关系：
   2.1. 不持有股票的状态转移:  
   
   ```cpp
   // 当天不持有股票的最大收益，认为是前一天不持有股票的最大收益，与前一天持有当卖出股票的最大收益之间的最大值。
   T[i][k][0] = max(T[i - 1][k][0], T[i - 1][k][1] + prices[i]);
   // 如果考虑有交易费用，后一项的最大收益需要减去交易费用
   T[i][k][0] = max(T[i - 1][k][0], T[i - 1][k][1] + prices[i] - fee);
   ```
   
   2.2. 持有股票的状态转移:
   
   ```cpp
   // 当天持有股票的最大收益，认为是前一天持有股票的最大收益，与前一天不持有而今天买入股票的最大收益
   T[i][k][1] = max(T[i - 1][k][1], T[i - 1][k - 1][0] - prices[i]);
   // 如果考虑交易费用，除了上面 2.1 在卖出时支付交易费用外，还可以考虑是在买入时支付交易费用
   T[i][k][1] = max(T[i - 1][k][1], T[i - 1][k - 1][0] - prices[i] - fee);
   // 如果考虑 cooldown 时间，那么我们能买入的条件也就不是前一天不持有股票，而是 cooldown 时间前不持有股票
   T[i][k][1] = max(T[i - 1][k][1], T[i - time][k - 1][0] - prices[i]);
   ```

#### 特殊 cases 的求解

有了上述的通用状态转移关系，我们可以比较轻松得求解参数值固定的情况。

##### case I : `k = 1`

当交易次数限制为 `k = 1` 时，每天 `i` 我们需要考虑两个状态下的值

```cpp
T[i][1][0] = max(T[i - 1][1][0], T[i - 1][1][1] + prices[i]);
T[i][1][1] = max(T[i - 1][1][1], T[i - 1][0][0] - prices[i]) = max(T[i - 1][1][1], - prices[i]);
```

从而，这道题的解法为：

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_10 = 0, T_11 = INT_MIN;
        for (int price : prices) {
            // 每次迭代最多只会修改其中一个值
            T_10 = max(T_10, T_11 + price);
            T_11 = max(T_11, -price);
        }
    }
    return T_10;
};
```

##### Case II: `k = INT_MAX`

这个时候，其实 `T[i-1][k-1][0] = T[i-1][k][0]` 且 `T[i-1][k-1][1] = T[i-1][k][1]`

每天 `i` ，我们仍然有两个变量未知

```cpp
T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])
T[i][k][1] = max(T[i-1][k][1], T[i-1][k-1][0] - prices[i]) = max(T[i-1][k][1], T[i-1][k][0] - prices[i])
```

从而，这道题的解法为：

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_k0 = 0, T_k1 = INT_MIN;
        for (int price : prices) {
            int T_k0_old = T_k0;
            T_k0 = max(T_k0, T_k1 + price);
            T_k1 = max(T_k1, T_k0_old - price);
        }
    }
    return T_k0;
};
```

##### Case III : `k = 2`

和 case I 很相似，不过现在每天需要考察四个变量，其状态转移关系为

```cpp
T[i][2][0] = max(T[i-1][2][0], T[i-1][2][1] + prices[i])
T[i][2][1] = max(T[i-1][2][1], T[i-1][1][0] - prices[i])
T[i][1][0] = max(T[i-1][1][0], T[i-1][1][1] + prices[i])
T[i][1][1] = max(T[i-1][1][1], - prices[i])
```

同理，解法为

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_10 = 0, T_11 = INT_MIN;
        int T_20 = 0, T_21 = INT_MIN;

        for (int price : prices) {
            T_20 = max(T_20, T_21 + price);
            T_21 = max(T_21, T_20 - price);
            T_10 = max(T_10, T_11 + price);
            T_11 = max(T_11, - price);
        }
    }
    return T_20;
};
```

通过这样的方法，我们可以解任意 k 的结果。

##### Case IV : `k = INT_MAX` 但是有冷却

我们考虑 cool down 为 1 的情况，也就是如果我们想要在第 `i` 天买股票，那么我们只能在第 `i - 2` 前将股票卖出。状态转移方程变为

```cpp
T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])
T[i][k][1] = max(T[i-1][k][1], T[i-2][k][0] - prices[i])
```

解法修改为

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_k0 = 0, T_k1 = INT_MIN, T_k0_pre = 0;
        for (int price : prices) {
            int T_k0_old = T_k0;
            T_k0 = max(T_k0, T_k1 + price);
            T_k1 = max(T_k1, T_k0_pre - price);
            T_k0_pre = T_k0_old;
        }
    }
    return T_k0;
};
```

##### Case V : `k = INT_MAX` 有交易费用

在 case II 的基础上，考虑购买或卖出时收益多减去一个 fee 即可

```cpp
T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])
T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i] - fee)
```

或者

```cpp
T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i] - fee)
T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i])
```

###### 解法 I : 买入时考虑费用

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_k0 = 0, T_k1 = INT_MIN;
        for (int price : prices) {
            int T_k0_old = T_k0;
            T_k0 = max(T_k0, T_k1 + price);
            T_k1 = max(T_k1, T_k0_old - price - fee);
        }
    }
    return T_k0;
};
```

###### 解法 II ：卖出时考虑费用

```cpp
class Solution {
  public:
    int maxProfit(vector<int>& prices) {
        int T_k0 = 0, T_k1 = INT_MIN;
        for (int price : prices) {
            int T_k0_old = T_k0;
            T_k0 = max(T_k0, T_k1 + price - fee);
            T_k1 = max(T_k1, T_k0_old - price);
        }
    }
    return T_k0;
};
```

##### 总结

对于这一类股票交易相关的问题，动态规划均可求解。

重点需要关注：

- 交易日期 `i`

- 最大允许交易次数 `k`

- 每天能有的状态

只要我们弄清楚每天这些状态之间的转移关系，就可以轻松 solve 这一系列问题。同时，弄清楚状态转移关系，还能对空间复杂度进行优化，不再需要使用一个数组去保留每天交易状态的最值。

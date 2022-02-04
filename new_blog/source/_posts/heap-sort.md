---
title: 堆排序代码实现
date: 2022-01-17 20:23:17
tags: 
- Leetcode
- Sort
- Heap
categories: 
- Leetcode
- Sort
cover: /img/HeapSort/heap_top.png
top_img: /img/HeapSort/heap_top.png
---

#### 什么是堆排序？

堆排序是一种基于比较的排序算法，需要借助于一种叫做二叉堆（Binary Heap）的数据结构。其操作和选择排序很相似，均可以认为是从某个数据结构中选出最小/最大的数出来。

##### 什么是二叉堆？

在定义二叉堆之前，让我们先声明一个满二叉树（完全二叉树）的定义。一个满二叉树是一个除最后一层以外，其他层都被完全占满的二叉树。同时，满二叉树还要求树的节点尽可能地靠近左边。

而二叉堆，则是一个有特殊排序的满二叉树，他每个父节点的值，都比两个子节点的值大(或者小)。前者，被称为最大堆，后者，则是最小堆。这样的堆我们可以用二叉树或者数组来表达。

##### 怎么用数组的格式表达二叉堆？

我们要充分利用好满二叉树每一层个数固定的性质。可以看到，如果一个父节点被存在了 `i` 位置，则其左子节点的位置为 `2 * i + 1`，而右子节点的位置为 `2 * i + 2`。这样的 array-base 的表达相比二叉树充分利用了空间。

#### Heapify 过程

将一个二叉树重构成一个堆数据结构的过程，被称作 heapify。

##### 增序的堆排序算法

步骤：

1. 通过输入数据件一个最大堆。

2. 最大的元素被放到了堆顶。用这个堆的最后一个值与堆顶交换，并使这个堆的 size 缩小一位。最终 heapify 树的根节点。

3. 重复第 2 步，知道堆只剩一个元素。

##### 代码实现

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

// heapify 一个根节点在 i 位置的子树，堆的大小为 n
void heapify(vector<int>& arr, int n, int i) {
    int largest = i;
    int l = 2 * i + 1;
    int r = 2 * i + 2;

    if (l < n && arr[l] > arr[largest]) largest = l;
    if (r < n && arr[r] > arr[largest]) largest = r;
    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}


// 堆排序过程

void heapSort(vector<int>& arr, int n) {
    // 0x01 建堆, 从第一个有子节点的节点开始 heapify
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }

    // 0x02 一个一个从 heap 中取出最大元素，放到最后，直到堆只剩一个元素
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);
        heapify(arr, i, 0);
    }
}


// test case
void printArray(vector<int>& arr) {
    for (int num : arr) std::cout << num << " ";
    std::cout << std::endl;
}


int main() {
    vector<int> arr = {17, 11, 13, 5, 3, 4};
    int n = arr.size();
    heapSort(arr, n);
    printArray(arr);
}
```

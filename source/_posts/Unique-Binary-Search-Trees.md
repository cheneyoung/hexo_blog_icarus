---
title: Unique Binary Search Trees
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
  - BST
abbrlink: 20d2cb13
date: 2018-11-16 00:42:32
---

**问题描述：**

Given n, how many structurally unique BST's (binary search trees) that store values 1...n?

For example,
Given n = 3, there are a total of 5 unique BST's.

```C++
   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

**解决思路：**

首先分析一下当n=0,1,2这三种情况下BTS个数，我们可以得到当n=0,1时，BTS个数为1，当n=2时，BTS个数为2.这就可以转化为一个递归求解的问题，我们将根结点从1到n依次代替，这样左右子树结点的个数就会发生变化，我们在分别求解左右子树BTS的个数，以此递归求解便可得到BTS的总数目。

**代码：**

```C++
class Solution {
public:
    int numTrees(int n) {
        if(n == 1 || n == 0)
            return 1;
        int num = 0;
        for(int i = 1; i <= n; i++)
            num += numTrees(i-1) * numTrees(n-i);
        return num;
    }
};
```


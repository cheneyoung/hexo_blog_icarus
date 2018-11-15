---
title: Single Number
comments: true
categories:
  - leetcode
tags:
  - C++
abbrlink: f249b7a1
date: 2018-11-16 00:49:48
---

**问题描述：**

Given an array of integers, every element appears twice except for one. Find that single one.

Note:
Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

**解决思路：**

本题主要是求出一组数组中唯一个没有相同元素的数字，这里要求时间复杂度为O(n)，空间复杂度为O(1)，所以我们只需遍历一次数组就必须得到结果，并且不能使用其他额外的内存空间。这里采用异或运算，能够保证相同的元素运算结果为0，所以最终便能找个那个唯一的数字。

**代码：**

```C++
class Solution {
 
public:
 
    int singleNumber(int A[], int n) {
 
        int result = 0;
 
        for(int i = 0;i < n; i++)
 
            result ^= A[i];
 
        return result;
 
    }
 
};
```


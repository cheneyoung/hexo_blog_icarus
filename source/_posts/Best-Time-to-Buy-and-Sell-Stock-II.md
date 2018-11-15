---
title: Best Time to Buy and Sell Stock II
comments: true
categories:
  - leetcode
tags:
  - C++
abbrlink: a10d34fe
date: 2018-11-16 00:48:29
---

**问题描述：**

Say you have an array for which the ith element is the price of a given stock on day i.

Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).

**解决思路：**

依次将相邻的元素相减（后面减去前面）得到新的数组，再取新数组正数相加即可

**代码：**

```C++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        if(prices.empty())
            return 0;
        vector<int> price_d(prices.size());
        for(int i = 0; i < prices.size()-1; i++)
        {
            price_d[i] = prices[i+1] - prices[i];
        }
        int max = 0;
        for(int i = 0; i < price_d.size(); i ++)
        {
            if(price_d[i] > 0)
                max += price_d[i];
        }
        return max;
    }
};
```


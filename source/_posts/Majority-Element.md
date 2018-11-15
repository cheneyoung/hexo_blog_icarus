---
title: Majority Element
comments: true
categories:
  - leetcode
tags: C++
abbrlink: ed8e0613
date: 2018-11-16 00:12:02
---

**问题描述：**

Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.

You may assume that the array is non-empty and the majority element always exist in the array.

**解决思路：**

这是一个求解数组主元素的题目，数组的主元素是指数组中出现次数超过一半的元素。此题是假设数组非空并且主元素存在，所以比较简单。主元素总会比其他元素多，我们设置两个变量maj_index用于记录主元素的位置，count用于记录主元素出现的次数，然后一次比较数组元素，如果相同则count加1，不同则减1，若count为零，则替换maj_index，并将count置为1.

**代码：**

```C++
class Solution {
public:
    int majorityElement(vector<int> &num) {
        int maj_index,count;
        maj_index = 0;
        count = 1;
        for(int i =1; i < num.size(); i++){
            if (num[i] == num[maj_index])
                count ++;
            else
                count --;
            if (count == 0){
                maj_index = i;
                count = 1;
            }
        }
        return num[maj_index];
    }
};
```


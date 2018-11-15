---
title: 3Sum
comments: true
categories:
  - leetcode
tags: C++
abbrlink: cc6b88ec
date: 2018-11-15 23:50:50
---

**问题描述：**

Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note:

Elements in a triplet (a,b,c) must be in non-descending order. (ie, a ≤ b ≤ c)
The solution set must not contain duplicate triplets.
​    For example, given array S = {-1 0 1 2 -1 -4},

    A solution set is:
    (-1, 0, 1)
    (-1, -1, 2)
**解决思路：**

3sum的时候，先整体排一次序，然后枚举第三个数字的时候不需要重复， 比如排好序以后的数字是 a b c d e f, 那么第一次枚举a, 在剩下的b c d e f中进行2 sum, 完了以后第二次枚举b, 只需要在 c d e f中进行2sum好了，而不是在a c d e f中进行2sum, 这个大家可以自己体会一下，想通了还是挺有帮助的。

```C++
class Solution {
public:
    vector<vector<int> > threeSum(vector<int> &num) {
        sort(num.begin(),num.end());
        int len = num.size();
        vector<vector<int>> result;
        if(len < 3)
            return result;
        
        int iCount = len - 2;
        vector<int> truple(3,0);
        int CurrentVal = num[0];
        
        for(int i = 0; i < iCount;i++)
        {
            if(i && num[i] == CurrentVal)
            {
                continue;
            }
            truple[0] = num[i];
            int l = i + 1;
            int r = len - 1;
            while(l<r)
            {
                int iSum = num[l] + num[r];
                if(iSum + truple[0] == 0)
                {
                    truple[1] = num[l];
                    truple[2] = num[r];
                    if(result.size() == 0 || truple != result[result.size()-1])
                        result.push_back(truple);
                    ++l;
                    --r;
                }
                else if(iSum + truple[0] < 0)
                {
                    ++l;
                }
                else
                {
                    --r;
                }
            }
            CurrentVal = num[i];
          
        }
        return result;
    }
    
};
```



这是一篇关于[kSum问题的总结性文章](http://tech-wonderland.net/blog/summary-of-ksum-problems.html)。
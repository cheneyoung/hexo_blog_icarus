---
title: 4Sum
comments: true
categories:
  - leetcode
tags: C++
abbrlink: 51bcb055
date: 2018-11-15 23:44:12
---

**问题描述**
Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.

Note: The solution set must not contain duplicate quadruplets.

For example, given array S = [1, 0, -1, 0, -2, 2], and target = 0.

A solution set is: 

```
[ 
[-1, 0, 0, 1], 
[-2, -1, 1, 2], 
[-2, 0, 0, 2] 
]
```

**算法1：O(n^3)**

我们可以仿照3sum的解决方法。这里枚举第一个和第二个数，然后对余下数的求2sum，算法复杂度为O（n^3）

代码：

```C++
class Solution {
public:
    vector<vector<int> > fourSum(vector<int> &num, int target) {
        int n = num.size();
        sort(num.begin(),num.end());
        vector<vector<int>> res;
        
        vector<int> quard(4,0);
        for(int i = 0; i < n-3; i++)
        {
            if(i > 0 && num[i] == num[i-1]) continue;
            for(int j = i + 1; j < n -2; j++)
            {
                if(j > i+1 && num[j] == num[j-1]) continue;
                int l = j+1,r=n-1;
                int target2 = target - num[i] - num[j];
                while(l<r)
                {
                    int sum = num[l] + num[r];
                    if(sum < target2)
                    {
                        ++l;
                    }
                    else if(sum > target2)
                    {
                        --r;
                    }
                    else
                    {
                        quard[0] = num[i];
                        quard[1] = num[j];
                        quard[2] = num[l];
                        quard[3] = num[r];
                        res.push_back(quard);
                        
                        int k = l + 1;
                        while(k < r && num[k] == num[l])k++;
                        l = k;
                        
                        k = r-1;
                        while(k > l && num[k] == num[r])k--;
                        r = k;
                        
                    }
                }
            }
        }
        return res;
    }
};
```



**算法2：O（n^2）的算法**

和前面相同，都是先对数组排序。我们先枚举出所有二个数的和存放在哈希map中，其中map的key对应的是二个数的和，因为多对元素求和可能是相同的值，故哈希map的value是一个链表（下面的代码中用数组代替），链表每个节点存的是这两个数在数组的下标；这个预处理的时间复杂度是O（n^2）。接着和算法1类似，枚举第一个和第二个元素，假设分别为v1,v2, 然后在哈希map中查找和为target-v1-v2的所有二元对（在对应的链表中），查找的时间为O（1），为了保证不重复计算，我们只保留两个数下标都大于V2的二元对（其实我们在前面3sum问题中所求得的三个数在排序后的数组中下标都是递增的），即时是这样也有可能重复：比如排好序后数组为-9 -4 -2 0 2 4 4，target = 0，当第一个和第二个元素分别是-4，-2时，我们要得到和为0-（-2）-（-4） = 6的二元对，这样的二元对有两个,都是(2,4)，且他们在数组中的下标都大于-4和-2，如果都加入结果，则(-4,-2,2,4)会出现两次，因此在加入二元对时，要判断是否和已经加入的二元对重复（由于过早二元对之前数组已经排过序，所以两个元素都相同的二元对可以保证在链表中是相邻的，链表不会出现(2,4)->(1,5)->(2,4)的情况，因此只要判断新加入的二元对和上一个加入的二元对是否重复即可），因为同一个链表中的二元对两个元素的和都是相同的，因此只要二元对的一个元素不同，则这个二元对就不同。我们可以认为哈希map中key对应的链表长度为常数，那么算法总的复杂度为O（n^2）

代码：

```c++
class Solution {
public:
    vector<vector<int> > fourSum(vector<int> &num, int target) {
        int n = num.size();
        vector<vector<int> > res;
        unordered_map<int, vector<pair<int, int> > >pairs;
        pairs.reserve(n*n);
        sort(num.begin(), num.end());
        
        for(int i = 0; i < n; i++)
            for(int j = i+1 ; j < n; j++)
                pairs[num[i]+num[j]].push_back(make_pair(i,j));
        
        for(int i = 0; i < n - 3; i++)
        {
            if(i != 0 && num[i] == num[i-1])continue;//防止第一个元素重复
            for(int j = i+1; j < n - 2; j++)
            {
                if(j != i+1 && num[j] == num[j-1])continue;//防止第二个元素重复
                if(pairs.find(target - num[i] - num[j]) != pairs.end())
                {
                    vector<pair<int, int>> &sum2 = pairs[target - num[i] - num[j]];
                    bool isFirstPush = true;
                    for(int k = 0; k < sum2.size(); k++)
                    {
                        if(sum2[k].first <= j)continue;//保证所求的四元组的数组下标是递增的
                        if(isFirstPush || (res.back())[2] != num[sum2[k].first])
                        {
                            res.push_back(vector<int>{num[i], num[j], num[sum2[k].first], num[sum2[k].second]});
                            isFirstPush = false;
                        }
                    }
                }
            }
        }
        
        return res;
    }
};
```


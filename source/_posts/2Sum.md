---
title: 2Sum
comments: true
categories:
  - leetcode
tags: C++
abbrlink: 74d7ef89
date: 2018-11-15 23:50:54
---

**问题描述：**

Given an array of integers, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

You may assume that each input would have exactly one solution.

Input: numbers={2, 7, 11, 15}, target=9
Output: index1=1, index2=2

**解决思路：**

本题主要是在给定的一个数组中找到两个数的下标，使得这个两个数和为给定的值，要求给出的下标从1 开始，且满足递增顺序，有三种解决方法，复杂度分别为O(n^2),O(nlogn),O(n),具体实现如下：



**复杂度为O(n^2) 代码：**

```C++
class Solution {
public:
    vector<int> twoSum(vector<int> &numbers, int target) {
        // Start typing your C/C++ solution below
        // DO NOT write int main() function
        vector<int> ret(2, 0);
        
        int len = numbers.size();
        
        for(int i = 0; i < len; i++)
        {
            int tmp = target - numbers[i];      // another number
            for(int j = i + 1; j < len; j++)
            {
                if(tmp == numbers[j])
                {
                    ret[0] = i + 1;     // +1 for not zero-based
                    ret[1] = j + 1;
                    return ret;
                }
            }
        }
        
        return ret;
    }
};
```



**复杂度O(nlogn)代码:**

```C++
class Solution {
public:
    vector<int> twoSum(vector<int> &numbers, int target) {
        vector<int> index;
        int len = numbers.size();
        vector<int> bak = numbers;
        
            
        sort(bak.begin(),bak.end());
        
        int l = 0,r = len-1;
        int sum = 0;
        while(l < r){
            sum = bak[l]+bak[r];
            if(sum == target)
            {
                for(int i = 0; i < len;i++)
                {
                    if (bak[l] == numbers[i])
                    {
                        index.push_back(i+1);
                    }
                    else if(bak[r] == numbers[i])
                    {
                        index.push_back(i+1);
                    }
                    if(index.size() == 2)
                    {
                        break;
                    }
                }
                break;
                
            }
            else if(sum < target)
            {
                ++l;
            }
            else
            {
                --r;
            }
            
        }
        
       return index;
    }
};
```



**复杂度O(n)代码:**

```C++
class Solution {
public:
    vector<int> twoSum(vector<int> &numbers, int target) {
        int len = numbers.size();  
        assert(len >= 2);             
          
        vector<int> ret(2, 0);  
          
        map<int, int> mapping;              // default all are 0  
        vector<long long> mul(len, 0);  
          
        for(int i = 0; i < len; i++){  
            mul[i] = (target - numbers[i]) * numbers[i];  
              
            if(mapping[mul[i]] > 0){        // not default 0  
                if(numbers[i] + numbers[mapping[mul[i]] - 1] == target){  
                    ret[0] = mapping[mul[i]];  
                    ret[1] = i + 1;  
                    break;  
                }  
                  
            } 
            else {  
                mapping[mul[i]] = i + 1;    // larger than 0  
            }  
        }  
          
        return ret;
    }
};
```


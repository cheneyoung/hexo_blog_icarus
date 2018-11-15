---
title: Excel Sheet Column Number
comments: true
categories:
  - leetcode
tags: C++
abbrlink: 50445eba
date: 2018-11-16 00:03:44
---

**问题描述：**

Related to question Excel Sheet Column Title

Given a column title as appear in an Excel sheet, return its corresponding column number.

For example:

```C++
A -> 1
B -> 2
C -> 3
...
Z -> 26
AA -> 27
AB -> 28 
```
**解决思路：**

本题类似于进制转换，将一个字符串转化为一个十进制的数字，而要求的字符串可以看作是一个26进制的数字。

**代码：**

```C++
class Solution {
public:
    int titleToNumber(string s) {
        if(s.empty())
            return 0;
        int n = 0;
        for(int i = 0; i < s.length()-1; i ++){
            n = (int(s[i]-'A')+1+n) * 26;
        }
        n += int(s[s.length()-1]-'A')+1;
        return n;
    }
};
```


---
title: Jewels-and-Stones
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - leetcode
tags:
  - C++
  - string
abbrlink: 57613
date: 2018-11-27 23:42:20
---

# 问题描述：

You're given strings `J` representing the types of stones that are jewels, and `S` representing the stones you have.  Each character in `S`is a type of stone you have.  You want to know how many of the stones you have are also jewels.

The letters in `J` are guaranteed distinct, and all characters in `J` and `S` are letters. Letters are case sensitive, so `"a"` is considered a different type of stone from `"A"`.

**Example 1:**

```shell
Input: J = "aA", S = "aAAbbbb"
Output: 3
```

**Example 2:**

```shell
Input: J = "z", S = "ZZ"
Output: 0
```

**Note:**

- `S` and `J` will consist of letters and have length at most 50.
- The characters in `J` are distinct.

# 解决思路：

此题要解决的问题是：统计字符串S中所有出现在字符串J中的字符个数



# AC代码：

- C++

```C++
class Solution {
public:
    int numJewelsInStones(string J, string S) {
        int len_S = S.length();
        int len_J = J.length();
        if (len_S == 0 || len_J == 0) return 0;
        int jewel_stone = 0;
        for (int i = 0; i < len_S; i++) {
            if (isInString(S[i],J)) {
                jewel_stone ++;
            }
        }
        return jewel_stone;
    }
    bool isInString(char target, string source) {
        for (int i = 0; i < source.length(); i++) {
            if (target == source[i]) {
                return true;
            } 
        }
        return false;
    } 
};
```

- python3

```python
class Solution:
    def numJewelsInStones(self, J, S):
        """
        :type J: str
        :type S: str
        :rtype: int
        """
        jewel_stone = 0
        if len(J) == 0 or len(S) == 0:
            return 0
        for i in range(len(S)):
            if S[i] in J:
                jewel_stone += 1
        return jewel_stone
```


---
title: ZigZag Conversion
comments: true
categories:
  - leetcode
tags: C++
abbrlink: 61984fa2
date: 2018-11-16 00:16:46
---

**问题描述：**

The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"
Write the code that will take a string and make this conversion given a number of rows:

string convert(string text, int nRows);
convert("PAYPALISHIRING", 3) should return "PAHNAPLSIIGYIR".

**解决思路：**

找出规律即可，每次的周期为2*nRows-2,但是在1到nRows-1的行数还需要判断周期为2*（nRows-1-i）的元素，i是处于1到nRows-1的行数。

**代码：**

```C++
class Solution {
public:
    string convert(string s, int nRows) {
        if(s.empty() || nRows <= 1 || s.length() < nRows)
            return s;
        string result;
        int cycle = 2 * nRows - 2;
        for(int i = 0; i < nRows; i++){
            for(int j = i; j < s.length(); j += cycle){
                result.push_back(s[j]);
                if( i > 0 && i < nRows-1){
                    if (j + 2 *(nRows - 1 - i) < s.length())
                        result.push_back(s[j + 2 *(nRows - 1 - i)]);
                }
            }
        }
        return result;
    }
};
```


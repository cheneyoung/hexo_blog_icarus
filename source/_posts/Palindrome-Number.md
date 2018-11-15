---
title: Palindrome Number
comments: true
categories:
  - leetcode
tags: C++
abbrlink: b216f26b
date: 2018-11-16 00:06:52
---

**问题描述：**

Determine whether an integer is a palindrome. Do this without extra space.

click to show spoilers.

Some hints:
Could negative integers be palindromes? (ie, -1)

If you are thinking of converting the integer to string, note the restriction of using extra space.

You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?

There is a more generic way of solving this problem.

**解决思路：**

这里要求不能使用额外的空间，基本思路是首先求出数据的位数，然后编写一个函数能够求出每一位的数字，然后遍历比较即可。

**代码1：**

```C++
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0)
            return false;
        int num = 0;
        int y = x;
        while( y != 0){
            num ++;
            y /= 10;
        }
        int i,j;
        for(i = 1, j = num; i < j;i++,j--){
            if (search(x,num,i) != search(x,num,j))
                return false;
        }
        return true;
    }
    int search(int x,int n,int index){
        int d = pow(10,n-index);
        int y = x/d;
        return y%10;
    }
};
```



**代码2：**

```C++
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0)
            return false;
        int z = x;
        int y = 0;
        while( x != 0){
            y *= 10;
            y += x%10;
            x /= 10;
        }
        if(z == y)
            return true;
        else
            return false;
    }
};
```


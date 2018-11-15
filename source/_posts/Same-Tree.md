---
title: Same Tree
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
  - DFS
abbrlink: f4467b0e
date: 2018-11-16 00:45:40
---

**问题描述：**

Given two binary trees, write a function to check if they are equal or not.

Two binary trees are considered equal if they are structurally identical and the nodes have the same value.

**解决思路：**

DFS遍历两颗二叉树的结点，只要有一个不同就返回false

```C++
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    bool isSameTree(TreeNode *p, TreeNode *q) {
        if(p == NULL)
        {
            if (q == NULL)
                return true;
            return false;
        }
        if(q == NULL)
        {
            if (p == NULL)
                return true;
            return false;
        }
        if (p->val != q->val)
            return false;
        bool lsame = isSameTree(p->left,q->left);
        bool rsame = isSameTree(p->right,q->right);
        if(lsame==false||rsame==false)
            return false;
        return true;
        
    }
};
```


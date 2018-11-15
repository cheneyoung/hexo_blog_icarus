---
title: Balanced Binary Tree
comments: true
categories:
  - leetcode
tags: 
  - C++ 
  - Binary Tree
abbrlink: 5c1c4dd0
date: 2018-11-16 00:14:50
---

**问题描述：**

Given a binary tree, determine if it is height-balanced.

For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.

**解决思路：**

首先设计一个求二叉树高度的函数，然后调用该函数比较左右子树的高度差即可。

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
    int height(TreeNode *root){
        if(root == NULL)
            return 0;
        int hr,hl;
        hl = height(root->left);
        hr = height(root->right);
        int h;
        h =1 + (hl > hr ? hl : hr);
        return h;
        
    }
    bool isBalanced(TreeNode *root) {
        if(root == NULL)
            return true;
        int hl = height(root->left);
        int hr = height(root->right);
        int diff = hl - hr;
        if(diff > 1 || diff < -1)
            return false;
        return isBalanced(root->left) && isBalanced(root->right);
    }
};
```


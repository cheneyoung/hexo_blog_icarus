---
title: Maximum Depth of Binary Tree
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
  - DFS
abbrlink: 2c2d1f13
date: 2018-11-16 00:47:11
---

**问题描述：**
Given a binary tree, find its maximum depth.

The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.

**解决思路：**

DFS分别求出左子树和右子树的最大深度然后比较即可

**代码：**

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
    int maxDepth(TreeNode *root) {
        if (root == NULL)
            return 0;
        int l = maxDepth(root->left);
        int r = maxDepth(root->right);
        return l>r?1+l:1+r;
        
    }
};
```
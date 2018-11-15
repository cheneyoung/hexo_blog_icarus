---
title: Binary Tree Inorder Traversal
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
abbrlink: 8a5e7d39
date: 2018-11-16 00:33:28
---

**问题描述：**

Given a binary tree, return the inorder traversal of its nodes' values.

For example:
Given binary tree {1,#,2,3},

   1
​    \
​     2
​    /
   3
return [1,3,2].

Note: Recursive solution is trivial, could you do it iteratively?

confused what "{1,#,2,3}" means? > read more on how binary tree is serialized on OJ.

**解决思路：**

本想用栈去实现，但是这上面好像没有栈的定义，没办法只好用vector去解决了。

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
    vector<int> inorderTraversal(TreeNode *root) {
        vector<int> intr;
        if (root == NULL)
            return intr;
        vector<TreeNode*> node;
        TreeNode *p;
        p = root;
        while(p != NULL || !node.empty()){
            if(p != NULL){
                node.push_back(p);
                p =p->left;
            }
            else{
                TreeNode *t = node.back();
                node.pop_back();
                intr.push_back(t->val);
                p = t->right;
        }
    }
    return intr;
    }
};
```
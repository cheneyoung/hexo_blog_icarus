---
title: Binary Tree Preorder Traversal
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
abbrlink: d13d2726
date: 2018-11-16 00:35:34
---

**问题描述：**

Given a binary tree, return the preorder traversal of its nodes' values.

For example:
Given binary tree {1,#,2,3},

   1
​    \
​     2
​    /
   3
return [1,2,3].

Note: Recursive solution is trivial, could you do it iteratively?

**非递归实现，不过超时了。**

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
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> pretr;
        if(root == NULL)
            return pretr;
        stack<TreeNode *> node;
        node.push(root);
        while(!node.empty()){
            TreeNode *p = node.top();
            pretr.push_back(p->val);
            node.pop();
            if(p->right) node.push(p);
            if(p->left) node.push(p);
        }
        return pretr;
    }
};
```

**递归实现**

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
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> pretr;
        process(root,pretr);
        return pretr;
        
    }
    void process(TreeNode *root,vector<int> &pretr){
        if(root == NULL) return;
        TreeNode *p;
        p = root;
        pretr.push_back(p->val);
        process(p->left,pretr);
        process(p->right,pretr);
    }
};
```


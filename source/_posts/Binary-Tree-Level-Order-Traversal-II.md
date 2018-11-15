---
title: Binary Tree Level Order Traversal II
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
abbrlink: 8adb33dd
date: 2018-11-16 00:40:06
---

**问题描述：**

Given a binary tree, return the bottom-up level order traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).

For example:
Given binary tree {3,9,20,#,#,15,7},

    	3
       / \
      9  20
        /  \
       15   7

return its bottom-up level order traversal as:

```
[
  [15,7],
  [9,20],
  [3]
]
```

confused what "{1,#,2,3}" means? > read more on how binary tree is serialized on OJ.


OJ's Binary Tree Serialization:
The serialization of a binary tree follows a level order traversal, where '#' signifies a path terminator where no node exists below.

Here's an example:

```
   1
  / \
 2   3
    /
   4
    \
     5
```

The above binary tree is serialized as "{1,2,3,#,#,4,#,#,5}".

**解决思路：**

和前面一样

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
    vector<vector<int> > levelOrderBottom(TreeNode *root) {
        vector<vector<int>> LOB;
        if (root == NULL)
            return LOB;
        int level = 0;
        int count = 1;
        queue<TreeNode *> node;
        node.push(root);
        stack<vector<int>> stk;
        vector<int> sub(0);
        while(!node.empty()){
            sub.clear();
            level = 0;
            for(int i = 0; i < count; i++){
                root = node.front();
                node.pop();
                sub.push_back(root->val);
                if(root->left != NULL){
                    node.push(root->left);
                    ++level;
                }
                if(root->right != NULL){
                    node.push(root->right);
                    ++level;
                }
            }
            stk.push(sub);
            count = level;
        }
        while(!stk.empty()){
            vector<int> tmp = stk.top();
            LOB.push_back(tmp);
            stk.pop();
        }
        return LOB;
    }
};
```


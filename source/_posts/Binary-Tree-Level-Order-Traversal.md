---
title: Binary Tree Level Order Traversal
comments: true
categories:
  - leetcode
tags:
  - C++
  - Binary Tree
abbrlink: a971a08f
date: 2018-11-16 00:37:35
---

**问题描述：**

Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).

For example:
Given binary tree {3,9,20,#,#,15,7},

     	3
       / \
      9  20
        /  \
       15   7

return its level order traversal as:

```
[
  [3],
  [9,20],
  [15,7]
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

这里需要用到队列，因为根据题意要求，我们的输出结果应该是包含vector的vector，里面的每个vector包含的是该层的所有节点的值。由于队列是先进先出的，所以首先建立一个节点队列，从根节点开始插入，另外用count记录每层节点的总个数，level计算每层节点的个数。

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
    vector<vector<int> > levelOrder(TreeNode *root) {
        vector<vector<int>> LO;
        if(root == NULL) return LO;
        queue<TreeNode *> node;
        node.push(root);
        int count = 1;
        int level = 0;
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
            count = level;
            LO.push_back(sub);
        }
        return LO;
    }
};
```


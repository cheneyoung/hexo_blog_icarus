---
title: Remove Duplicates from Sorted List
comments: true
categories:
  - leetcode
tags: 
  - C++
  - List
abbrlink: 1d8b65b1
date: 2018-11-16 00:18:19
---

**问题描述：**

Given a sorted linked list, delete all duplicates such that each element appear only once.

For example,
Given 1->1->2, return 1->2.
Given 1->1->2->3->3, return 1->2->3.

**解决思路：**

定义两个指针base和cmp,base指针指向被比较的结点，cmp指向base的后一个结点，由于链表是有序的，所以cmp找到第一个不等于base的结点之后，就分别将base和cmp后移。如果相等，只要将相等的结点删除即可。

**代码:**

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *deleteDuplicates(ListNode *head) {
        if(head == NULL || head->next == NULL)
            return head;
        ListNode *base,*cmp;
        base = head;
        cmp = base->next;
        while(cmp != NULL){
            ListNode *tmp;
            tmp = cmp->next;
            if(cmp->val == base->val){
                base->next = tmp;
                free(cmp);
                cmp = tmp;
            }
            else{
                base = cmp;
                cmp = tmp;
            }
            
        }
        return head;
        
    }
};
```


---
title: Linked List Cycle
comments: true
categories:
  - leetcode
tags:
  - C++
  - List
abbrlink: c94099d8
date: 2018-11-16 00:20:05
---

**问题描述：**

Given a linked list, determine if it has a cycle in it.

Follow up:
Can you solve it without using extra space?

**解决思路：**

采用快慢指针的方法，快指针一次走两步，慢指针一次走一步，如果链表中有环的话，经过走一定的步数之后快慢指针一定会相遇的，当然如果没有环就需考虑循环结束的条件，这里主要要考虑快指针的情况即可。

**代码：**

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
    bool hasCycle(ListNode *head) {
        if(head == NULL)
            return false;
        if(head->next == NULL)
            return false;
        ListNode *p,*q;
        p = head;
        q = head;
       while(p != NULL && p->next != NULL)
        {
            p = p->next->next;
            q = q->next;
            if(p == q)
                return true;
            
        }
        return false;
    }
};
```


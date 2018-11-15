---
title: Intersection of Two Linked Lists
comments: true
categories:
  - leetcode
tags: 
  - C++
  - List
abbrlink: 42870ae4
date: 2018-11-16 00:09:39
---

**问题描述：**

For example, the following two linked lists:

A:          a1 → a2
​                   ↘
​                     c1 → c2 → c3
​                   ↗            
B:     b1 → b2 → b3
begin to intersect at node c1.


Notes:

If the two linked lists have no intersection at all, return null.
The linked lists must retain their original structure after the function returns.
You may assume there are no cycles anywhere in the entire linked structure.
Your code should preferably run in O(n) time and use only O(1) memory.

**解决思路：**

这题要求的时间复杂度是O(n)和空间复杂度O(1)，所以只能用以下方法解决：(1)首先分别求出两个单链表的长度lenA和lenB；(2)求出两个链表的长度差，然后将长链表的比较位置移到长度差的位置，因为这些位置上的两个链表的元素是肯定不相同的；(3)此时，两个链表的长度一致，在分别依次比较元素即可

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
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if(headA == NULL || headB == NULL)
            return NULL;
        int lenA,lenB;
        lenA = 0,lenB = 0;
        ListNode *p,*q;
        for(p = headA;p != NULL;p=p->next)
            lenA ++;
        for(q = headB;q != NULL;q=q->next)
            lenB ++;
        int diff = abs(lenB-lenA);
        if(lenA > lenB){
            p = headA;
            for(int i = 0;i < diff;i++)
                p = p->next;
            q = headB;
            while(p != NULL && q != NULL){
                if(p->val == q->val)
                    return p;
                else{
                    p = p->next;
                    q = q->next;
                }
            }
            return NULL;
        }
        else{
            p = headB;
            for(int i = 0;i < diff;i++)
                p = p->next;
            q = headA;
            while(p != NULL && q != NULL){
                if(p->val == q->val)
                    return p;
                else{
                    p = p->next;
                    q = q->next;
                }
            }
            return NULL;
        }
    }
};
```


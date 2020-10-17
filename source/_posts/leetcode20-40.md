---
title: leetcode20-40
tags:
  - LeetCode
  - Algorithm
mathjax: true
date: 2018-01-04 18:22:08
---

# 21. Merge Two Sorted Lists

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

<!-- more -->

```
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```

## Python

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None
############ iteration #################
class Solution(object):
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        temp=ans=ListNode(0)
        while(l1 and l2):
            v1 = l1.val
            v2 = l2.val
            if v1<v2:
                temp.next = ListNode(v1)
                l1 = l1.next
                temp=temp.next
            else:
                temp.next = ListNode(v2)
                l2 = l2.next
                temp=temp.next
        if l1:
            temp.next = l1
        else:
            temp.next=l2
        return ans.next
 ############ recursion #################
class Solution(object):
    def mergeTwoLists(self, l1, l2):
		"""
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if l1 is None:
            return l2
        elif l2 is None:
            return l1
        elif l1.val<l2.val
        	l1.next = mergeTwoLists(l1.next,l2)
            return l1
        else:
            l2.next = mergeTwoLists(l1,l2.next)
            return l2

```

## C++

```c++

```

# 
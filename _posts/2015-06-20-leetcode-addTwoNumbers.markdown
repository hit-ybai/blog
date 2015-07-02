---
layout: post
title: "LeetCode: Add Two Numbers 解题报告"
date: 2015-06-20 00:10:27 +0800
comments: true
categories: 解题报告
tags:
- leetcode
- linked list
---
[题目描述](https://leetcode.com/problems/add-two-numbers/):
*You are given two linked lists representing two non-negative numbers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.*

*Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)*
*Output: 7 -> 0 -> 8*

---
分析：对两个链表进行带进位求和，若长度不一致填零

题解：没什么算法，模拟。
```cpp
#include <cstddef>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *ans_pointer = NULL, *ans_head = NULL;
        
        int sum = 0;
        bool carry_flag = false;
        while (true) {
            sum = getSum(getNodeVal(l1), getNodeVal(l2), carry_flag);

            if (ans_pointer) {
                ans_pointer->next = new ListNode(sum);
                ans_pointer       = ans_pointer->next;
            } else {
                ans_head    = new ListNode(sum);
                ans_pointer = ans_head;
            }
            nextPointer(l1);
            nextPointer(l2);
            if ( !(l1 || l2) ) {
                if (carry_flag) { ans_pointer->next = new ListNode(1); }
                break;
            }
        }
        return ans_head;
    }
private:
    int getNodeVal(const ListNode* const node) const {
        return node ? node->val : 0;
    }
    
    int getSum(const int val1, const int val2, bool &carry_flag) const {
        int sum = val1 + val2 + carry_flag;
        carry_flag = (sum >= 10);
        return sum % 10;
    }
    
    void nextPointer(ListNode* &node) const {
        node = node ? node->next : NULL;
    }
};
```
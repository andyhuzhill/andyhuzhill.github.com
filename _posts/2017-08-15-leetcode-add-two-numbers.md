---
layout: post
title: "LeetCode #2 Add Two Numbers"
description: ""
category: algorithm
tags: [algorithm, leetcode]
---
{% include JB/setup %}

## 题目

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Input**: (2 -> 4 -> 3) + (5 -> 6 -> 4)
**Output**: 7 -> 0 -> 8

*翻译:*
给出两个非空链表表示的两个非负整数。返回值为一个链表表示的两个数的和。

## 解答

这一题没有什么难度的，直接计算加法即可，需要注意的是进位和空节点的处理。

### 头节点不为空
{% highlight cpp %}
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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int carry = 0;
        ListNode *head = nullptr;
        ListNode *tail = nullptr;
        
        while ((l1 != nullptr) || (l2 != nullptr)) {
            int left = 0;
            int right = 0;
            if (l1 != nullptr) {
                left = l1->val;
                l1 = l1->next;
            }
            if (l2 != nullptr) {
                right = l2->val;
                l2 = l2->next;
            }
            int sum = (left + right + carry);
            carry = sum / 10;
            if (head == nullptr) {
                tail = new ListNode(sum % 10);
                head = tail;
            } else {
                tail->next = new ListNode(sum % 10);
                tail = tail->next;
            }
        } 

        if (carry != 0) {
            tail->next = new ListNode(carry);
        }
        return head;
    }
};
{% endhighlight %}

### 头节点为空
{% highlight cpp %}
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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int carry = 0;
        ListNode head(0);
        ListNode *tail = &head;
        
        while ((l1 != nullptr) || (l2 != nullptr)) {
            int left = (l1 != nullptr) ? l1->val : 0;
            int right = (l2 != nullptr) ? l2->val : 0;
            if (l1 != nullptr) {
                l1 = l1->next;
            }
            if (l2 != nullptr) {
                l2 = l2->next;
            }
            int sum = (left + right + carry);
            carry = sum / 10;
            tail->next = new ListNode(sum % 10);
            tail = tail->next;
        } 

        if (carry != 0) {
            tail->next = new ListNode(carry);
        }
        return head.next;
    }
};
{% endhighlight %}
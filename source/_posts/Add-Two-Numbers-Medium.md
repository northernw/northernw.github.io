---
title: Add Two Numbers | Medium
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-15 17:48:27
---
leetCode: [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

## Description
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.

## Solution

### First
嗯，裹脚布，又臭又长
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int carry = 0;
        ListNode head = null;
        ListNode ans = new ListNode(0);
        while (l1 != null && l2 != null) {
            int sum = l1.val + l2.val + carry;
            if (sum >= 10) {
                carry = 1;
                sum %= 10;
            } else {
                carry = 0;
            }
            ans.next = new ListNode(sum);
            ans = ans.next;
            if (head == null) {
                head = ans;
            }

            l1 = l1.next;
            l2 = l2.next;
        }

        while (l1 != null) {
            int sum = carry + l1.val;
            if (sum >= 10) {
                carry = 1;
                sum %= 10;
            } else {
                carry = 0;
            }
            ans.next = new ListNode(sum);
            ans = ans.next;
            l1 = l1.next;
        }

        while (l2 != null) {
            int sum = carry + l2.val;
            if (sum >= 10) {
                carry = 1;
                sum %= 10;
            } else {
                carry = 0;
            }
            ans.next = new ListNode(sum);
            ans = ans.next;
            l2 = l2.next;
        }

        if (carry > 0) {
            ans.next = new ListNode(carry);
        }

        return head;
    }
}

```

### Better
ref: (a 11-line cpp solution)[https://leetcode.com/problems/add-two-numbers/discuss/997/c%2B%2B-Sharing-my-11-line-c%2B%2B-solution-can-someone-make-it-even-more-concise]

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int carry = 0;
        ListNode ans = new ListNode(0);
        ListNode head = ans;
        ListNode zero = new ListNode(0);
        while (l1 != null || l2 != null || carry > 0) {
            if (l1 == null) {
                l1 = zero;
            }
            if (l2 == null) {
                l2 = zero;
            }
            int sum = l1.val + l2.val + carry;
            carry = sum / 10;
            ans.next = new ListNode(sum % 10);
            ans = ans.next;

            l1 = l1.next;
            l2 = l2.next;
        }

        return head.next;
    }
}
```

## Given Solution
```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```

## Note

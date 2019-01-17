---
title: Reverse Integer | Easy | leetCode
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-14 14:58:20
---
leetCode: [7. Reverse Integer](https://leetcode.com/problems/reverse-integer/)

## Description

Given a 32-bit signed integer, reverse digits of an integer.

Example 1:

Input: 123
Output: 321
Example 2:

Input: -123
Output: -321
Example 3:

Input: 120
Output: 21
Note:
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [-2^31,  2^31 - 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.


## Solution

### First
30ms
```
class Solution {
    public int reverse(int x) {
        int res = 0;
        int tmp = x;
        boolean negtive = x < 0;
        if(negtive){
            tmp = -tmp;
        }
        while(tmp != 0){
            int carry = tmp % 10;
            if(res > (Integer.MAX_VALUE - carry)/10){
                return 0;
            }
            res = res *10 + carry;
            tmp /= 10;
        }
        return negtive ? -res : res;
    }
}

```

## Note
overflow溢出
Integer的最大值MAX_VALUE，最小值MIN_VALUE

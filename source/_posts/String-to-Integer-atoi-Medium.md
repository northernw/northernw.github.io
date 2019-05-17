---
title: String to Integer (atoi) | Medium
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-23 16:43:49
---
leetCode: [8. String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)

## Description
```
Implement atoi which converts a string to an integer.

The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.

The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.

If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.

If no valid conversion could be performed, a zero value is returned.

Note:

Only the space character ' ' is considered as whitespace character.
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. If the numerical value is out of the range of representable values, INT_MAX (231 − 1) or INT_MIN (−231) is returned.
Example 1:

Input: "42"
Output: 42
Example 2:

Input: "   -42"
Output: -42
Explanation: The first non-whitespace character is '-', which is the minus sign.
             Then take as many numerical digits as possible, which gets 42.
Example 3:

Input: "4193 with words"
Output: 4193
Explanation: Conversion stops at digit '3' as the next character is not a numerical digit.
Example 4:

Input: "words and 987"
Output: 0
Explanation: The first non-whitespace character is 'w', which is not a numerical 
             digit or a +/- sign. Therefore no valid conversion could be performed.
Example 5:

Input: "-91283472332"
Output: -2147483648
Explanation: The number "-91283472332" is out of the range of a 32-bit signed integer.
             Thefore INT_MIN (−231) is returned.

```


## Solution

### First
不是很喜欢这类题
但是巧妙的解法还是很多啊

ref: [leetcode discuss](https://leetcode.com/problems/string-to-integer-atoi/discuss/4654/My-simple-solution)

```
class Solution {
    public int myAtoi(String str) {
        int sign = 1, i = 0, ans = 0, n=str.length();
        if(n==0){
            return 0;
        }
		// 这里的while和if都很巧妙
        while(i<n && str.charAt(i)==' '){
            i++;
        }
        if(i<n && (str.charAt(i)=='-' || str.charAt(i)=='+')){
            sign = 1 - 2*(str.charAt(i)=='-'?1:0);
            i++;
        }
        int max = Integer.MAX_VALUE/10;
		// 这部分while直接忽略掉后续的不合法字符，也很赞
		// 边界判断尾数>'7'，对正负数都有效
		// "-2147483648"  '8'>'7'，正好返回最小值
        while(i<n && str.charAt(i)>='0' && str.charAt(i)<='9'){
            if(ans>max || (ans == max && str.charAt(i)>'7')){
                return sign==1?Integer.MAX_VALUE:Integer.MIN_VALUE;
            }
            ans = ans * 10 + (str.charAt(i)-'0');
            i++;
        }
        return sign*ans;
    }
}

```

## Note

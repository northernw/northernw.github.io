---
title: 9. Palindrome Number | Easy
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-23 17:11:28
---
leetCode: [9. Palindrome Number](https://leetcode.com/problems/palindrome-number/)

## Description
```
Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

Example 1:

Input: 121
Output: true
Example 2:

Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
Example 3:

Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
Follow up:

Coud you solve it without converting the integer to a string?

```


## Solution

### First

笨法子
Runtime: 103 ms, faster than 68.27% of Java online submissions for Palindrome Number.
```
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0){
            return false;
        }
        StringBuilder sb = new StringBuilder();
        while(x > 0){
            sb.append(x%10);
            x/=10;
        }
        String s = sb.toString();
        int n = s.length();
        for(int i=0;i<n/2;i++){
            if(s.charAt(i)!=s.charAt(n-i-1)){
                return false;
            }
        }
        return true;
    }
}

```

## Recomended

```
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0 || (x!=0 && x%10==0)){
            return false;
        }
        int reverse = 0;
        while(x>reverse){
            reverse = reverse * 10 + x % 10;
            x/=10;
        }
        return (x == reverse || x==reverse/10);
    }
}

```

## Note

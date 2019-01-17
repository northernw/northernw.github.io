---
title: Longest Palindromic Substring | Medium
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-17 11:15:12
---
leetCode: [5. Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

## Description

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

Example 1:

Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
Example 2:

Input: "cbbd"
Output: "bb"

## Solution

### First
原想暴力破解，想了下时间复杂度还是太高，看了hints决定提笔算。
做的题都忘光光了，加油！

145ms, faster than 20.90% of Java online submissions.
仍需努力呀

Time complexity: O(n^2)
Space complexity: O(n^2)
```
class Solution {
    public String longestPalindrome(String s) {
        // Dynamic Programming
        int n = s.length();
        boolean[][] flag = new boolean[n][n];
        
        for(int len = 0; len < n; len++){
            for(int i = 0; i< n-len; i++){
                int start = i, end = i + len;
                if((start == end) || (start+1 == end)){
                    flag[start][end] = s.charAt(start) == s.charAt(end);
                }else{
                    flag[start][end] = (s.charAt(start) == s.charAt(end)) && flag[start+1][end-1];
                }
            }
        }
        
        int len = 0;
        String ans = "";
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                if(flag[i][j] && (j-i+1 > len)){
                    len = j-i+1;
                    ans = s.substring(i,j+1);
                }
            }
        }
        
        return ans;
    }
}

```

优化了下，ans不在flag循环中取，还是145ms，大头仍是O(n^2)
```
        
        int max = 0;
        int left = 0, right = 0;
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                if(flag[i][j] && (j-i+1 > max)){
                    max = j-i+1;
                    left = i;
                    right = j;
                }
            }
        }
        
        return s.substring(left,right+1);
```

## Recomended

最早想的方案也是这个，怎么就没想到有2n-1个中心呢
没留意到偶数处理方案

16 ms, faster than 93.37% of Java online submissions.
赞~
时间复杂度还是O(n^2)

Expand Around Center
```
class Solution {
    private int left = 0;
    private int len = 0;
    private int n = 0;
    
    public String longestPalindrome(String s) {
        n = s.length();
        if(n < 2){
            return s;
        }
        
        for(int i = 0; i < n; i++){
            aroundCenter(s, i, i);
            aroundCenter(s, i, i+1);
        }
        
        return s.substring(left, left+len);
    }
    
    private void aroundCenter(String s, int start, int end){
        while(start>=0 && end <n && s.charAt(start)==s.charAt(end)){
            start--;
            end++;
        }
		// 此时start+1和end-1为正确下标，回文子串长度为(end-1)-(start+1)+1
        if(end - start -1 > len){
            left = start + 1;
            len = end - start -1;
        }
    }
}
```

## Compare
做了个简单的对比
输入字符串长约900字符
平均 中心点法1ms，DP 20ms+

原想对比dp两次for循环会增加多少时间，貌似不太准，作罢
```
@Test
public void longestPalindrome() throws Exception {
	LongestPalindromicSubstring longestPalindromicSubstring = new LongestPalindromicSubstring();
	long start = System.currentTimeMillis();
	System.out.println(longestPalindromicSubstring.longestPalindromeWithAroundCenter(s));
	System.out.println(System.currentTimeMillis() - start);

	start = System.currentTimeMillis();
	System.out.println(longestPalindromicSubstring.longestPalindromeWithDP(s));
	System.out.println(System.currentTimeMillis() - start);
}
```

## Note
* dp找到状态转移方程，简单理解为递推公式
* 数组下标的计算，对数字不太敏感，笨
	* s[i,j], i与j中有j-i+1个元素
	* s[i]走j-i步到s[j]，因为s[i]本身占一个元素，再过j-i个元素
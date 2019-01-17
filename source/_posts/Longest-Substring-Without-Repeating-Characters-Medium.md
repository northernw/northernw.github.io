---
title: Longest Substring Without Repeating Characters | Medium
tags:
  - leetCode
categories:
  - leetCode
date: 2019-01-15 20:59:49
---
leetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

## Description
Given a string, find the length of the longest substring without repeating characters.

Example 1:

Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
Example 2:

Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
Example 3:

Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

## Solution

### First
Time Limit Exceeded

```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int ans = 1, len = s.length();
        if(len == 0){
            return 0;
        }
        Set<Character> set = new HashSet();
        for(int left = 0; left < len-1; left++){
            for(int right = left+1; right < len; right++){
                set.clear();
                for(int i = left; i < right+1; i++){
                    set.add(s.charAt(i));
                }            
                if(right-left+1 == set.size() && set.size() > ans){
                    ans = set.size();
                }
            }
        }
        return ans;        
    }
}

```

## Recommended

Sliding Window
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        // sliding window
        int n = s.length();
        int ans = 0;
        Set<Character> set = new HashSet<>();
        for(int i = 0, j = 0; i < n && j < n; ){
            // try to extend the range of [i, j]
            if(!set.contains(s.charAt(j))){
                ans = Math.max(ans, j - i +1);
                set.add(s.charAt(j++));
            }else{
                set.remove(s.charAt(i++));
            }
        }
        return ans;
    }
}
```

Sliding Window Optimized
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        // sliding window optimized
        int n = s.length();
        int ans = 0;
        // current index of character
        Map<Character,Integer> map = new HashMap<>();
        for(int i = 0, j = 0; j < n; j++){
            // try to extend the range of [i, j]
            if(map.containsKey(s.charAt(j))){
                i = Math.max(i, map.get(s.charAt(j)));
            }
            ans = Math.max(ans, j - i + 1);
            map.put(s.charAt(j), j + 1);
        }
        return ans;
    }
}
```

## Note
* String substring(int,int) 左开右闭截取
* charAt(int) 取char
* Set<Character> set = new HashSet<>();
* 滑动窗解法，s[i,j)视为满足条件的子串，比较s[j]与s[i,j)，若满足条件，j++，若不满足条件，i++

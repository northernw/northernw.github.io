---
title: Two Sum | Easy | leetCode
date: 2019-01-14 11:23:30
tags: 
- leetCode
categories:
- leetCode
---
leetCode: [1.Two Sum](https://leetcode.com/problems/two-sum/)

## Description
Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

## Solution

### First
45ms
```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int len = nums.length;
        int i=0,j=0;
        for(;i<len;i++){
            for(j=i+1; j<len;j++){
                if(nums[i]+nums[j] == target){
                    return new int[]{i,j};
                }
            }
        }
        return null;
    }
}
```

### Better
7ms
```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map = new HashMap<Integer,Integer>();
        for(int i = 0; i < nums.length; i++){
            if(map.containsKey(target - nums[i])){
                return new int[]{map.get(target - nums[i]),i};
            }
            map.put(nums[i],i);
        }
        return new int[]{0,0};
    }
}
```

### Faster
说是6ms.. 记录下
```
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map = new HashMap<Integer,Integer>();
        for(int i = 0; i < nums.length; i++){
            if(map.get(target - nums[i])!=null){
                return new int[]{map.get(target - nums[i]),i};
            }
            map.put(nums[i],i);
        }
        return new int[]{0,0};
    }
}
```

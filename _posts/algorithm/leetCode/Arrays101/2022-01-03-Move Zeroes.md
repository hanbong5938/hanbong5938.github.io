---
layout : post
title : "Move Zeroes"
category : LeetCode[Arrays 101]
---

* Source : https://leetcode.com/explore/learn/card/fun-with-arrays/511/in-place-operations/3157/
* Author : Han
* Date   : 2022-01-03

### Problems

Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.

**Note** that you must do this in-place without making a copy of the array.


**Example 1**
```
Input: nums = [0,1,0,3,12]
Output: [1,3,12,0,0]
```

**Example 2**
```
Input: nums = [0]
Output: [0]
```

###Solution
```java
class Solution {
    public void moveZeroes(int[] nums) {
        final int length = nums.length;
        int zeroCount = 0;
        
        // 0 카운트 및 땅기기
        for(var i = 0; i < length; i++) {
            if (nums[i] == 0) {
                zeroCount++;
            } else{
                nums[i - zeroCount] = nums[i];
            }
        }
        // 0 투입
        for (var i = 0; i < zeroCount; i++) {
            nums[length - zeroCount + i] = 0;
        }
    }
}
```

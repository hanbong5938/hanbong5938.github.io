---
layout : post 
title : "Find All Numbers Disappeared in an Array"
category : LeetCode[Arrays 101]
---

* Source : [Link](https://leetcode.com/explore/learn/card/fun-with-arrays/523/conclusion/3270/)
* Author : Han
* Date   : 2022-01-06

### Problems
Given an array nums of n integers where nums[i] is in the range [1, n], return an array of all the integers in the range [1, n] that do not appear in nums.

**Example 1**

```
Input: nums = [4,3,2,7,8,2,3,1]
Output: [5,6]
```

**Example 2**

```
Input: nums = [1,1]
Output: [2]
```

### Solution

```kotlin
class Solution {
    fun findDisappearedNumbers(nums: IntArray): List<Int> {
        return  nums.let { num -> (1..nums.size).filter { it !in num} }
    }
}
```
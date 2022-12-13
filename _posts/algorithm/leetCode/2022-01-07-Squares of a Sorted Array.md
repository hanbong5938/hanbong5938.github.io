---
layout : post 
title : "Squares of a Sorted Array"
category : algorithm[LeetCode]
---

* Source : [Link](https://leetcode.com/explore/learn/card/fun-with-arrays/523/conclusion/3574/)
* Author : Han
* Date   : 2022-01-07

### Problems
Given an integer array `nums` sorted in **non-decreasing** order, return *an array of **the squares of each number** sorted in non-decreasing order*.

**Example 1:**

```
Input: nums = [-4,-1,0,3,10]
Output: [0,1,9,16,100]
Explanation: After squaring, the array becomes [16,1,0,9,100].
After sorting, it becomes [0,1,9,16,100].
```

**Example 2:**

```
Input: nums = [-7,-3,2,3,11]
Output: [4,9,9,49,121]
```

### Solution

```kotlin
import kotlin.math.pow

class Solution {
    fun sortedSquares(nums: IntArray): IntArray {
        return nums.map { it.toDouble().pow(2).toInt() }.sorted().toIntArray()
    }
}
```
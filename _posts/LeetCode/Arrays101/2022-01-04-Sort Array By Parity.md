---
layout : post
title : "Sort Array By Parity"
category : LeetCode[Arrays 101]
---

* Source : [Link](https://leetcode.com/explore/learn/card/fun-with-arrays/511/in-place-operations/3260/)
* Author : Han
* Date   : 2022-01-04

### Problems

Given an integer array `nums`, move all the even integers at the beginning of the array followed by all the odd integers.

Return any array that satisfies this condition.

**Example 1**
```
Input: nums = [3,1,2,4]
Output: [2,4,3,1]
Explanation: The outputs [4,2,3,1], [2,4,1,3], and [4,2,1,3] would also be accepted.
```

**Example 2**
```
Input: nums = [0]
Output: [0]
```

###Solution
```java
class Solution {
    public int[] sortArrayByParity(int[] nums) {
        final int length = nums.length;
        for(int i = 0; i < length; i++) {
            if((nums[i] % 2) == 1){
                for(int j = i + 1; j < length; j++){
                    if ((nums[j] % 2) == 0) {
                        final int temp = nums[j];
                        nums[j] = nums[i];
                        nums[i] = temp;
                    }
                }
            }
        }
        return nums;
    }
}
```

**It's better solution**
```java
class Solution {
    public int[] sortArrayByParity(int[] nums) {
        final int two = 2;
        return Arrays.stream(nums).boxed().sorted((a, b) -> Integer.compare(a % two, b % two))
                .mapToInt(i -> i).toArray();
    }
}
```
---
layout : post
title : "Replace Elements with Greatest Element on Right Side"
category : algorithm[LeetCode]
---

* Source : https://leetcode.com/explore/learn/card/fun-with-arrays/511/in-place-operations/3259/
*  Author : Han
*  Date   : 2022-01-03

### Problems

Given an array arr, replace every element in that array with the greatest element among the elements to its right,
and replace the last element with -1.

After doing so, return the array.

**Example 1**
```
Input: arr = [17,18,5,4,6,1]
Output: [18,6,6,6,1,-1]
Explanation: 
- index 0 --> the greatest element to the right of index 0 is index 1 (18).
- index 1 --> the greatest element to the right of index 1 is index 4 (6).
- index 2 --> the greatest element to the right of index 2 is index 4 (6).
- index 3 --> the greatest element to the right of index 3 is index 4 (6).
- index 4 --> the greatest element to the right of index 4 is index 5 (1).
- index 5 --> there are no elements to the right of index 5, so we put -1.
```

**Example 2**
```
Input: arr = [400]
Output: [-1]
Explanation: There are no elements to the right of index 0.
```

###Solution
```java
class Solution {
    public int[] replaceElements(int[] arr) {
        final int size = arr.length;
        final int[] result = new int[size];
        for(int i = 0; i < size; i++) {
            int temp = -1;
            for (int j = i + 1; j < size; j++) {
                final int num = arr[j];
                if (num > temp) {temp = num;}
            }
            result[i] = temp;
        }
        return result;
    }
}
```

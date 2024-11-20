# 33. Search in Rotated Sorted Array

LeetCode URL: https://leetcode.com/problems/search-in-rotated-sorted-array/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

解き方を覚えていたので書けはしました。

- TODO

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    private final int NOT_FOUND = -1;

    public int search(int[] nums, int target) {
        if (nums == null || nums.length == 0) {
            throw new IllegalArgumentException("nums must not be null or empty");
        }

        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            }

            if (nums[left] <= nums[middle]) {
                if (target < nums[left] || nums[middle] < target) {
                    left = middle + 1;
                } else {
                    right = middle - 1;
                }
            } else {
                if (target < nums[middle] || nums[right] < target) {
                    right = middle - 1;
                } else {
                    left = middle + 1;
                }
            }
        }
        return NOT_FOUND;
    }
}
```
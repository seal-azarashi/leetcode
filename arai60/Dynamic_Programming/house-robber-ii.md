# 213. House Robber II

LeetCode URL: https://leetcode.com/problems/house-robber-ii/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
/**
 * 時間計算量: O(n):
 *     - O(n): 配列の先頭から末尾の要素を除いた全ての要素を対象に robHelper() を実行
 *     - O(n): 配列2つ目の要素から末尾まで全ての要素を対象に robHelper() を実行
 * 空間計算量: O(1): 各走査で必要な変数等定数量の追加空間
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }

        return Math.max(
            robHelper(0, nums.length - 1, nums),
            robHelper(1, nums.length, nums)
        );
    }

    private int robHelper(int start, int end, int[] nums) {
        int twoBackMaxCount = 0, oneBackMaxCount = 0;
        for (int i = start; i < end; i++) {
            int maxCount = Math.max(twoBackMaxCount + nums[i], oneBackMaxCount);

            twoBackMaxCount = oneBackMaxCount;
            oneBackMaxCount = maxCount;
        }
        return oneBackMaxCount;
    }
}
```

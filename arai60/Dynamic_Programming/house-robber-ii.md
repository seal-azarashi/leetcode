# 213. House Robber II

LeetCode URL: https://leetcode.com/problems/house-robber-ii/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

House Robber の拡張版のような問題。今回は家が円状に並んでおり、最初と最後の家が隣接する。  
隣接する最初と最後の家は同時に盗みに入れないため、配列から最初 or 最後の家を除いた計2つの配列を作り、それぞれで盗めるお金の最大値から、より大きいものが答えになる。  
(💭 ヘルパーメソッドの中身をどうするかについては [House Robber のプルリクエスト](https://github.com/seal-azarashi/leetcode/pull/33)で議論がされていますので、そちらも合わせてご覧いただけますと嬉しいです)  

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

## Step 2

他の方のプルリクエストに目を通したが、注目すべき事項はほぼ [House Robber の方](https://github.com/seal-azarashi/leetcode/pull/33)を解く際に把握出来ていたようなので、今回は割愛。

## Step 3

```java
/**
 * 解いた時間: 約4分
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
        int twoBackMaxAmount = 0, oneBackMaxAmount = 0;
        for (int i = start; i < end; i++) {
            int maxAmount = Math.max(twoBackMaxAmount + nums[i], oneBackMaxAmount);

            twoBackMaxAmount = oneBackMaxAmount;
            oneBackMaxAmount = maxAmount;
        }
        return oneBackMaxAmount;
    }
}
```

## Step 4

[oda さんのレビュー](https://github.com/seal-azarashi/leetcode/pull/34#discussion_r1791578173)を踏まえ関数名を修正。

```java
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }

        return Math.max(
            maxRobbableAmountInRange(0, nums.length - 1, nums),
            maxRobbableAmountInRange(1, nums.length, nums)
        );
    }

    private int maxRobbableAmountInRange(int start, int end, int[] nums) {
        int twoBackMaxAmount = 0, oneBackMaxAmount = 0;
        for (int i = start; i < end; i++) {
            int maxAmount = Math.max(twoBackMaxAmount + nums[i], oneBackMaxAmount);

            twoBackMaxAmount = oneBackMaxAmount;
            oneBackMaxAmount = maxAmount;
        }
        return oneBackMaxAmount;
    }
}
```

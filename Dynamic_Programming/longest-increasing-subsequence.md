# 300. Longest Increasing Subsequence

LeetCode URL: https://leetcode.com/problems/longest-increasing-subsequence/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

計算量 O(2^n) になる Brute force な方法しか思いつかなかったので、答えを見て書きました。

```java
/**
 * 時間計算量: O(n^2):
 *     - O(n): キャッシュ用配列の作成
 *     - O(n^2): 各要素とそれ以降のキャッシュすべてを利用した sequence 算出処理
 *     - O(n): キャッシュから longestIncreasingSubsequence を探す処理
 * 空間計算量: O(n)
 *     - O(n): 引数 nums と同じサイズのキャッシュ用配列
 */
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] sequenceLengthCache = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            sequenceLengthCache[i] = 1;
        }
        for (int i = sequenceLengthCache.length - 1; i >= 0; i--) {
            for (int j = i + 1; j < sequenceLengthCache.length; j++) {
                if (nums[i] < nums[j]) {
                    sequenceLengthCache[i] = Math.max(sequenceLengthCache[i], 1 + sequenceLengthCache[j]);
                }
            }
        }
        int longestIncreasingSubsequence = 1;
        for (int i = 0; i < sequenceLengthCache.length; i++) {
            longestIncreasingSubsequence = Math.max(longestIncreasingSubsequence, sequenceLengthCache[i]);
        }
        return longestIncreasingSubsequence;
    }
}
```

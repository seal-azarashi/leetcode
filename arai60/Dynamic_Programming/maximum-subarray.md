# 53. Maximum Subarray

LeetCode URL: https://leetcode.com/problems/maximum-subarray/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

前に解いていたが解答を忘れてたので、当時見ていたであろうビデオを見て復習しながら書きました: https://www.youtube.com/watch?v=5WZl3MMT0Eg

```java
// 時間計算量: O(n): 引数 nums の要素すべて走査する
// 空間計算量: O(1): 固定サイズの変数が決まった個数宣言される
class Solution {
    public int maxSubArray(int[] nums) {
        int maxSubArray = nums[0];
        int sumInWindow = 0;
        for (int num : nums) {
            if (sumInWindow < 0) {
                sumInWindow = 0;
            }
            sumInWindow += num;
            maxSubArray = Math.max(maxSubArray, sumInWindow);
        }
        return maxSubArray;
    }
}
```

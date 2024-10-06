# 198. House Robber

LeetCode URL: https://leetcode.com/problems/house-robber/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

最初に思いついた、ある地点での最大 amount of money をキャッシュしていく実装。

```java
/**
 * かかった時間: 約15分
 * 時間計算量: O(n): 
 *     - O(n): キャッシュ用配列の宣言 (引数に受け取る配列のコピー)
 *     - O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(n):
 *     - O(n): キャッシュ用配列
 *     - O(1): return する結果を格納する変数
 */
class Solution {
    public int rob(int[] nums) {
        int[] amountCache = Arrays.copyOfRange(nums, 0, nums.length);
        int maxAmount = 0;
        for (int i = 0; i < amountCache.length; i++) {
            if (i == 2) {
                amountCache[i] += amountCache[i - 2];
            } else if (2 < i) {
                amountCache[i] += Math.max(amountCache[i - 3], amountCache[i - 2]);
            }
            maxAmount = Math.max(maxAmount, amountCache[i]);
        }
        return maxAmount;
    }
}
```

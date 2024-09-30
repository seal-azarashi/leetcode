# 300. Longest Increasing Subsequence

LeetCode URL: https://leetcode.com/problems/longest-increasing-subsequence/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

計算量 O(2^n) になる Brute force な方法しか思いつかなかったので、 DP の解法を見て書きました。  
変数名について、 longestIncreasingSubsequence と書くと長すぎて読みづらくなったので lis と略称を使いましたが、こうして見ると結構な改善の余地ありですね。  

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
        int[] lis = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            lis[i] = 1;
        }
        for (int i = lis.length - 1; i >= 0; i--) {
            for (int j = i + 1; j < lis.length; j++) {
                if (nums[i] < nums[j]) {
                    lis[i] = Math.max(lis[i], 1 + lis[j]);
                }
            }
        }
        int answer = 1;
        for (int i = 0; i < lis.length; i++) {
            answer = Math.max(answer, lis[i]);
        }
        return answer;
    }
}
```

## Step 2

### DP (Step 1 のブラッシュアップ)

- 変数名 lis -> sequenceLengthCache に変更
    - 入っているものは sequence の length なのでそれがわかるようにした
        - lis は longest increasing subsequence の略だが、 longest な要素は一つしかないので配列名としては不適当だった
        - 昇順シーケンスなので increasing があった方が適当ではあるが、ここで明記が無くても関数名等合わせれば自然に分かると判断し、読みやすさのために削った
    - キャッシュに使っている事を明示した
    - nodchip さんのレビューを参考にしました: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536793230
- 二重 for loop 内のインデックス名を修正: i -> currentIndex, j -> subsequentIndex
    - それぞれ何を指すインデックスなのか、ループの内容を把握しないと理解出来なく読みづらいため
    - nodchip さんのレビューを参考にしました: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536792178

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
        for (int currentIndex = sequenceLengthCache.length - 1; currentIndex >= 0; currentIndex--) {
            for (int subsequentIndex = currentIndex + 1; subsequentIndex < sequenceLengthCache.length; subsequentIndex++) {
                if (nums[currentIndex] < nums[subsequentIndex]) {
                    sequenceLengthCache[currentIndex] = Math.max(
                        sequenceLengthCache[currentIndex],
                        1 + sequenceLengthCache[subsequentIndex]
                    );
                }
            }
        }
        int longestIncreasingSubsequence = 1;
        for (int i = 0; i < sequenceLengthCache.length; i++) {
            longestIncreasingSubsequence = Math.max(
                longestIncreasingSubsequence,
                sequenceLengthCache[i]
            );
        }
        return longestIncreasingSubsequence;
    }
}
```

### セグメントツリー

エンジニア常識範囲外の解法とのことなので一旦保留: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536794621

# 139. Word Break

LeetCode URL: https://leetcode.com/problems/word-break/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

最初 brute force な解法しか思いつかなかったので、以下の通りの実装をしました。TLE。

```java
/**
 * ※ n = s の長さ、m = wordDict の要素数
 * 時間計算量: O(m^n * m):
 *     - O(m^n): s から生成される部分文字列の数 (最大で wordDict の要素数ごとに n だけ生成される)
 *     - O(m): イテレーションごとの wordDict の走査
 * 空間計算量: O(m^n): s から生成される部分文字列の数 (最大で wordDict の要素数ごとに n だけ生成される)
 */
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Queue<String> remainStringQueue = new ArrayDeque<>();
        remainStringQueue.offer(s);
        while (!remainStringQueue.isEmpty()) {
            String remainString = remainStringQueue.poll();
            for (String word : wordDict) {
                if (!remainString.startsWith(word)) {
                    continue;
                }

                String nextString = remainString.replaceFirst(word, "");
                if (nextString.equals("")) {
                    return true;
                }
                remainStringQueue.offer(nextString);
            }
        }
        return false;
    }
}
```

Dynamic Programming の問題なのでどこかの値をキャッシュ出来るよな... と推測してしまい、そこから次のような、どの位置を起点とすれば、それ以降の部分文字列がセグメンテーション可能なのかをキャッシュする実装を思いつきました。

```java
/**
 * ※ n = s の長さ、m = wordDict の要素数
 * 時間計算量: O(n * (n + m * n)):
 *     - O(n): s から生成される部分文字列の数
 *         - O(n): 部分文字列の生成処理
 *         - O(m): イテレーションごとの wordDict の走査
 *             - O(n): 部分文字列と辞書の単語の比較
 *     - O(1): その他の処理
 * 空間計算量: O(n): セグメンテーションの可能位置を保持する配列
 */
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        boolean[] canBeSegmentedFrom = new boolean[s.length()];
        for (int i = s.length() - 1; i >= 0; i--) {
            String substring = s.substring(i, s.length());
            for (String word : wordDict) {
                if (!substring.startsWith(word)) {
                    continue;
                }

                int nextWordHead = i + word.length();
                if (nextWordHead >= s.length() || canBeSegmentedFrom[nextWordHead]) {
                    canBeSegmentedFrom[i] = true;
                }
            }
        }
        return canBeSegmentedFrom[0];
    }
}
```

上の実装から向きを変え、どの位置を起点とすれば、それ**以前**の部分文字列がセグメンテーション可能なのかをキャッシュする実装もしてみました。  
Java の substring() メソッドが半開 (左閉右開) で区間を指定する都合上、少しマジックナンバー感が出てしまって読みづらくなった印象です。

```java
/**
 * ※ n = s の長さ、m = wordDict の要素数
 * 時間計算量: O(n * (n + m * n)):
 *     - O(n): s から生成される部分文字列の数
 *         - O(n): 部分文字列の生成処理
 *         - O(m): イテレーションごとの wordDict の走査
 *             - O(n): 部分文字列と辞書の単語の比較
 *     - O(1): その他の処理
 * 空間計算量: O(n): セグメンテーションの可能位置を保持する配列
 */
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        boolean[] canBeSegmentedTo = new boolean[s.length()];
        for (int i = 0; i < s.length(); i++) {
            String substring = s.substring(0, i + 1);
            for (String word : wordDict) {
                if (!substring.endsWith(word)) {
                    continue;
                }

                int previousWordTail = i - word.length();
                if (previousWordTail < 0 || canBeSegmentedTo[previousWordTail]) {
                    canBeSegmentedTo[i] = true;
                }
            }
        }
        return canBeSegmentedTo[s.length() - 1];
    }
}
```

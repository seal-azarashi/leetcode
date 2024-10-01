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

### 配列の末尾からある要素以降のシーケンス長を算出する (Step 1 のブラッシュアップ)

- 変数名 lis -> sequenceLengthCache に変更
    - 入っているものは sequence の length なのでそれがわかるようにした
        - lis は longest increasing subsequence の略だが、 longest な要素は一つしかないので配列名としては不適当だった
        - 昇順シーケンスなので increasing があった方が適当ではあるが、ここで明記が無くても関数名等合わせれば自然に分かると判断し、読みやすさのために削った
    - キャッシュに使っている事を明示した
    - nodchip さんのレビューを参考にしました: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536793230
- 二重 for loop 内のインデックス名を修正: i -> currentIndex, j -> subsequentIndex
    - それぞれ何を指すインデックスなのか、ループの内容を把握しないと理解出来なく読みづらいため
    - nodchip さんのレビューを参考にしました: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536792178
- Arrays に fill() というメソッドがあったので使ってみる

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
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int[] sequenceLengthCache = new int[nums.length];
        Arrays.fill(sequenceLengthCache, 1);
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

### 対応するインデックスの要素で終わる最長増加部分列の長さを配列に入れる

他の方も悩まれてましたが、配列と関数の命名が難しいですね... これも直感的かどうかあんまり自信ないので、ご意見頂けますと嬉しいです。

```java
/**
 * 時間計算量: O(n^2):
 *     - O(n): キャッシュ用配列の作成
 *     - O(n^2): 各要素とそれ以降の特定要素までの最長増加部分列算出処理
 *     - O(n): キャッシュから longestIncreasingSubsequence を探す処理
 * 空間計算量: O(n)
 *     - O(n): 引数 nums と同じサイズのキャッシュ用配列
 */
class Solution {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int[] maxLengthAsTail = new int[nums.length];
        Arrays.fill(maxLengthAsTail, 1);
        for (int i = 1; i < nums.length; i++) {
            maxLengthAsTail[i] = findMaxLengthWithNewTail(nums, maxLengthAsTail, i);
        }

        int longestIncreasingSubsequence = 1;
        for (int i = 0; i < maxLengthAsTail.length; i++) {
            longestIncreasingSubsequence = Math.max(longestIncreasingSubsequence, maxLengthAsTail[i]);
        }
        return longestIncreasingSubsequence;
    }

    private int findMaxLengthWithNewTail(int[] nums, int[] maxLengthAsTail, int tailIndex) {
        int maxLength = 1;
        for (int i = 0; i < tailIndex; i++) {
            if (nums[i] < nums[tailIndex]) {
                maxLength = Math.max(maxLength, maxLengthAsTail[i] + 1);
            }
        }
        return maxLength;
    }
}
```

### 二分探査を用いた O(n log n) の解法

Java の標準ライブラリの二分探索メソッド、返り値がどうなるか理解するのがしんどかった... 要素が見つからなかったことを示すために負の値にするのはいいんだけど、なんで挿入位置を 1-indexed の値で返すようにしたんだろう🤔

```java
/**
 * 時間計算量: O(n log n): nums の要素それぞれに対して最大 (log nums.length) の二分探索
 * 空間計算量: O(n): 引数 nums と同じサイズの配列
 */
class Solution {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int[] minLISTailValues = new int[nums.length];
        int lisLength = 0;
        for (int num : nums) {
            int insertPosition = Arrays.binarySearch(minLISTailValues, 0, lisLength, num);
            if (insertPosition < 0) {
                // 要素が見つからない場合、挿入位置を計算
                // 参考: https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#binarySearch-int:A-int-int-int-
                insertPosition = -(insertPosition + 1);
            }
            minLISTailValues[insertPosition] = num;
            if (insertPosition == lisLength) {
                lisLength++;
            }
        }
        return lisLength;
    }
}
```

可変長配列使ったほうが直感的ですね。 Leetcode 上では処理時間が2倍ほどになってましたが、桁が違うわけではないのでよっぽどシビアなパフォーマンス要件がない限りはこっちを選びたいです (というかそれだけシビアにするなら言語を変えるところから検討した方がよさそう)。  
こちらも命名が難しいですが、こちらの議論を参考に実施してみました: https://github.com/Yoshiki-Iwasa/Arai60/pull/46/files/56e8cf4d4efc42c5784108191d1e5fc615de9206#r1716128766

```java
/**
 * 時間計算量: O(n log n): nums の要素それぞれに対して最大 (log nums.length) の二分探索
 * 空間計算量: O(n): 最大で引数 nums と同じサイズになる可変長配列
 */
class Solution {
    public int lengthOfLIS(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        // is: increasing subsequence
        ArrayList<Integer> isMinTailValues = new ArrayList<>();
        for (int num : nums) {
            int insertPosition = Collections.binarySearch(minISTailValues, num);
            if (insertPosition < 0) {
                // 要素が見つからない場合、挿入位置を計算
                // 参考: https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#binarySearch-java.util.List-T-
                insertPosition = -(insertPosition + 1);
            }
            if (insertPosition == minISTailValues.size()) {
                minISTailValues.add(num);
            } else {
                minISTailValues.set(insertPosition, num);
            }
        }
        return minISTailValues.size();
    }
}
```

### 自分で二分探索ロジックを書く

TODO: Arai60 一通り終えたら実装 (その頃には別の問題を通して書けるようになっているはず...?)

### BIT を用いた解法

TODO: 放送大学では見なかったが、レビュー見る限り常識範囲外というわけでもないらしいのでこちらも後で理解してみる。

### セグメントツリー

エンジニア常識範囲外の解法とのことなので一旦保留: https://github.com/shining-ai/leetcode/pull/31#discussion_r1536794621

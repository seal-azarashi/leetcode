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

## Step 2

### Kadane's Algorithm (Step 1 の解法のブラッシュアップ)

- 『なっとくアルゴリズム』を勧めてくださってた pco2699 さんの記事を参考に少し修正しました: https://blog.pco2699.net/entry/2020/05/15/225011
- Constraints 上不正となる引数が渡されてもエラーにならないようチェックを入れました
- 変数名が適当でないと気づいたので修正しました: maxSubArray -> maxSubArraySum

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = nums[0];
        int currentSum = 0;
        for (int num : nums) {
            currentSum = Math.max(num, num + currentSum);
            maxSubArraySum = Math.max(maxSubArraySum, currentSum);
        }
        return maxSubArraySum;
    }
}
```

### Cubic な解法

タイムアウトするようなコードも書けるようになっておけばそこからより良い解法を考えられるようになる、とどこかで oda さんが仰ってたのを思い出して書いてみました。予想通り TLE。

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = nums[0];
        for (int i = 0; i < nums.length; i++) {
            for (int j = i; j < nums.length; j++) {
                int subArraySum = 0;
                for (int k = i; k <= j; k++) {
                    subArraySum += nums[k];
                }
                maxSubArraySum = Math.max(maxSubArraySum, subArraySum);
            }
        }
        return maxSubArraySum;
    }
}
```

### Squared な解法

Cubic な解法と同様のモチベーションで書きました。 TLE。

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = nums[0];
        for (int i = 0; i < nums.length; i++) {
            int currentSum = 0;
            for (int j = i; j < nums.length; j++) {
                currentSum += nums[j];
                maxSubArraySum = Math.max(maxSubArraySum, currentSum);
            }
        }
        return maxSubArraySum;
    }
}
```

### Divide and conquer

Leetcode に follow up として "try coding another solution using the divide and conquer approach" とあったので、分割統治法でも書いてみた。

```java
TODO
```

### みんなのプルリク見た結果

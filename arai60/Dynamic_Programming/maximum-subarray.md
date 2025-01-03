# 53. Maximum Subarray

LeetCode URL: https://leetcode.com/problems/maximum-subarray/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

前に解いていたが解答を忘れてたので、当時見ていたであろうビデオを見て復習しながら書きました: https://www.youtube.com/watch?v=5WZl3MMT0Eg

```java
/**
 * 時間計算量: O(n): 引数 nums の要素すべて走査する
 * 空間計算量: O(1): 固定サイズの変数が決まった個数宣言される
 */
class Solution {
    public int maxSubArray(int[] nums) {
        int maxSubArray = Integer.MIN_VALUE;
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
/**
 * 時間計算量: O(n): 引数 nums の要素すべて走査する
 * 空間計算量: O(1): 固定サイズの変数が決まった個数宣言される
 */
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = Integer.MIN_VALUE;
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
/**
 * 時間計算量: O(n^3):
 *     - O(n): 配列の全要素を走査
 *         - O(n): 操作中の要素以降の全要素を走査
 *             - O(n): 最大で配列と同じ数の要素の合計を算出
 * 空間計算量: O(1): 計算に必要ないくつかの変数を宣言
 */
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = Integer.MIN_VALUE;
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
/**
 * 時間計算量: O(n^2):
 *     - O(n): 配列の全要素を走査
 *         - O(n): 操作中の要素以降の全要素を走査して最大値を更新
 * 空間計算量: O(1): 計算に必要ないくつかの変数を宣言
 */
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = Integer.MIN_VALUE;
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

Kadane's Algorithm はエンジニアの常識でもないし、問題見ただけで一発で思いつくのはあまりにも天才的すぎると [Discord でコメント](https://discord.com/channels/1084280443945353267/1206101582861697046/1207405733667410051)がありましたが、自分もこんなのは思いつかないので、暗記しておかない限りは面接で出せるとしたらこの解答になるだろうなという印象です。  

また上記のコメントを一通り読んで、思いついた解法があるのなら、それをベストなものでないと切り捨ていきなり解答を見るようなことはせず、書いてみてブラッシュアップさせていくような作業をしないとなと思いました。  

> 15分の使い方として、「かっこいい公式を連想してみせよう」とかやっているのは、あまり意味がなくて、とりあえず、いいから手を動かしてくれと思っています。  
> かっこいい公式を思いついたからそれをどうやって振り回すかを考えている15分もあんまり意味がないという感覚です。  
> [-2,1,-3,4,-1,2,1,-5,4]を例に取れば、100回ちょっとも足し算すれば、全通り出せるわけで、とりあえずそれをやってから考えたらどうでしょう。  
> from: https://discord.com/channels/1084280443945353267/1206101582861697046/1207949240316338176

> (Kadane's Algorithm のような解法を思いつく過程の例を示した上で)
> 
> 東京から新宿まで移動してくれと言われたら、中央線もあれば、丸ノ内線もあれば、タクシーを拾ってもいいし、自分で運転してもいいし、歩いていってもいいじゃないですか。これくらいの幅を持って見ていて、その中には愚直なのもあれば、短いのもあれば、速いのもあれば、遅いのもあり、色々なバランスを見て、今日はこれくらいにしておくかな、くらいの感覚で選んでいます。  
> 上で並べたのは、「下に行けば行くほど洗練されていて無条件に良い」などではなくて、だいたいこれくらいの幅を持って見ているということを伝えたかったからです。  
> どうせ、1番上は思いついたけれども、価値のないものだと思って捨てたでしょう。そこがいかんのですよ。  
> 時間あるし運動もしたいから2時間くらい歩くか、という選択肢を持っているかどうかで、対応がだいぶ違います。都内の地理に詳しい人から聞いた結論だけ暗記してもあんまり意味はないのです。  
> from: https://discord.com/channels/1084280443945353267/1206101582861697046/1208473290881110117

> (上記コメントの補足として)
>
> - 大事なのは、見たときに大局的に色々な手段が見えていること。  
> - その中には遅いものも速いものもあり、色々な良し悪しで評価できること。例えば速度の見積もりとかもそれ。  
> - それぞれの手段の間の移り変わりの関係性が見えていること。  
> - 局所的に変更して、見やすくしたり、整理したりすることができること。  
> 
> だいたい、この辺です。移り変わり、みたいなものを上で特に表現してみました。  
> from: https://discord.com/channels/1084280443945353267/1206101582861697046/1209027377397506109

### Divide and conquer

Leetcode に follow up として "try coding another solution using the divide and conquer approach" とあったので、分割統治法でも書いてみた。

```java
/**
 * 時間計算量: O(n log n):
 *     - O(1): 配列の分割
 *     - O(log n): 再帰の深さ (配列を半分ずつ分割するため)
 *         - O(n): 各再帰処理における計算量 (中央を跨ぐ maxCrossingSubArray() の計算)
 *         - O(1): 結合処理 (leftMax, rightMax, crossMax の最大値の算出)
 * 空間計算量: O(log n):
 *     - O(log n): スタックフレームの数 (再帰の深さ)
 *     - O(1): 計算に用いる変数
 */ 
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        return maxSubArrayHelper(nums, 0, nums.length - 1);
    }

    private int maxSubArrayHelper(int[] nums, int left, int right) {
        if (left > right) {
            throw new IllegalArgumentException(
                String.format("不正な範囲指定: left=%d > right=%d", left, right)
            );
        }

        if (left == right) {
            return nums[left];
        }

        // left < right のとき、必ず left <= mid < right
        int mid = left + (right - left) / 2;
        int leftMax = maxSubArrayHelper(nums, left, mid);
        int rightMax = maxSubArrayHelper(nums, mid + 1, right);
        int crossMax = maxCrossingSubArray(nums, left, mid, right);
        return Math.max(Math.max(leftMax, rightMax), crossMax);
    }

    private int maxCrossingSubArray(int[] nums, int left, int mid, int right) {
        if (left > mid || mid >= right) {
            throw new IllegalArgumentException(
                String.format("不正な範囲指定: 条件 left <= mid < right を満たしていません: left=%d, mid=%d, right=%d",
                    left, mid, right)
            );
        }

        int leftSum = Integer.MIN_VALUE;
        int currentLeftSum = 0;
        for (int i = mid; i >= left; i--) {
            currentLeftSum += nums[i];
            leftSum = Math.max(leftSum, currentLeftSum);
        }

        int rightSum = Integer.MIN_VALUE;
        int currentRightSum = 0;
        for (int i = mid + 1; i <= right; i++) {
            currentRightSum += nums[i];
            rightSum = Math.max(rightSum, currentRightSum);
        }

        return leftSum + rightSum;
    }
}
```

## Step 3

```java
/**
 * 解いた時間: 2~3分
 * 時間計算量: O(n): 引数 nums の要素すべて走査する
 * 空間計算量: O(1): 固定サイズの変数が決まった個数宣言される
 */
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = Integer.MIN_VALUE;
        int currentSum = 0;
        for (int num : nums) {
            currentSum = Math.max(num, num + currentSum);
            maxSubArraySum = Math.max(maxSubArraySum, currentSum);
        }
        return maxSubArraySum;
    }
}
```

## Step 4

[こちらの指摘](https://github.com/seal-azarashi/leetcode/pull/30/files/286d89eec55442db67a108dd6622799403618234#r1789292251)に対応。複数該当箇所があるが、代表して step 3 の実装の修正版を書く。

```java
/**
 * 時間計算量: O(n): 引数 nums の要素すべて走査する
 * 空間計算量: O(1): 固定サイズの変数が決まった個数宣言される
 */
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int maxSubArraySum = Integer.MIN_VALUE;
        int subArraySumSoFar = 0;
        for (int num : nums) {
            subArraySumSoFar = Math.max(num, num + subArraySumSoFar);
            maxSubArraySum = Math.max(maxSubArraySum, subArraySumSoFar);
        }
        return maxSubArraySum;
    }
}
```

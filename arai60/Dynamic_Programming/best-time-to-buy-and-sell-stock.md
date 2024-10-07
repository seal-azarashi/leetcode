# 121. Best Time to Buy and Sell Stock

LeetCode URL: https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

ぱっと綺麗な解答が思いつかなかったので、まずは愚直に全パターンを算出する O(n^2) の計算量の実装をする。 TLE。

```java
/**
 * かかった時間: 約4分
 * 時間計算量: O(n^2):
 *     - O(n): 配列の全ての要素を走査
 *         - O(n): 走査中の要素以降全ての要素を走査
 *             - O(1): 最大利益の計算
 * 空間計算量: O(1): ある時点での最大利益を格納する変数
 */
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null) {
            return 0;
        }

        int maxProfit = 0;
        for (int i = 0; i < prices.length - 1; i++) {
            for (int j = i + 1; j < prices.length; j++) {
                maxProfit = Math.max(maxProfit, prices[j] - prices[i]);
            }
        }
        return maxProfit;
    }
}
```

全パターンを算出するのは無駄な気がしてならないので、もっと効率の良い処理を考える。まず、買い日と売り日は前後関係を維持し続けなければならない。なのでそれぞれ別のポインタで管理し、特定の条件ごとに前にだけ進めていく中で最大利益を更新していけば、配列を一度走査するだけで解答が導けると考えた。  
結局、買い日と売り日それぞれ2つのポインタを使って管理し、各イテレーションで次の処理を行うようにした:  

- 利益が出る限り最大利益の確認をし、売り日のポインタを進める
- 損益が出たら買い日のポインタを売り日のポインタに移し、その翌日を新たな売り日とする

(💭 一度、損益が出た際に買い日と売り日のポインタを一つずつ後ろにずらすように実装しましたが、これだと買い日と売り日の感覚がリセットされずに期待通りの挙動にならなかったため wrong answer となりました。カッコいいコードが書けなかったどころか不正解となりましたが、これを書いたおかげで頭の中が整理され、解答を導くことが出来ました。頭の中で考えているだけだともっと時間がかかったと思うので、とりあえず書き出すのは大事ですね。)

```java
/**
 * かかった時間: 約18分
 * 時間計算量: O(n):
 *     - O(n): 配列の全ての要素を走査
 *         - O(1): 最大利益の確認
 *         - O(1): ポインタの操作
 * 空間計算量: O(1): ある時点での最大利益を格納する変数、買い日と売り日のポインタ
 */
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null) {
            return 0;
        }

        int maxProfit = 0;
        int buy = 0, sell = 1;
        while (sell < prices.length) {
            int profit = prices[sell] - prices[buy];
            if (profit < 0) {
                buy = sell;
                sell = buy + 1;
                continue;
            }

            maxProfit = Math.max(maxProfit, profit);
            sell++;
        }
        return maxProfit;
    }
}
```

## Step 2

### 買い日と売り日のポインタを用いる (Step 1 のブラッシュアップ)

- prices のインデックスは売り or 買いの日を表すので、それぞれの変数名に `~Day` と付けた
- while 文だと中身を把握するまでポインタの動きがイメージ出来ないため、ある程度それが想像しやすい for 文に書き直した

```java
/**
 * 時間計算量: O(n):
 *     - O(n): 配列の全ての要素を走査
 *         - O(1): 最大利益の確認
 *         - O(1): ポインタの操作
 * 空間計算量: O(1): ある時点での最大利益を格納する変数、買い日と売り日のポインタ
 */
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null) {
            return 0;
        }

        int maxProfit = 0;
        int buyDay = 0;
        for (int sellDay = 1; sellDay < prices.length; sellDay++) {
            int profit = prices[sellDay] - prices[buyDay];
            if (profit < 0) {
                buyDay = sellDay;
                continue;
            }
            maxProfit = Math.max(maxProfit, profit);
        }
        return maxProfit;
    }
}
```

### i 日目の価格 - (i - 1) 日目までの最小の価格を計算して最大利益を出す

上記とは違う考え方で解かれた [hayashi-ay さんの解答](https://github.com/hayashi-ay/leetcode/pull/52/files)を参考に書いてみる。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) {
            return 0;
        }

        int maxProfit = 0;
        int minPriceSoFar = prices[0];
        for (int i = 1; i < prices.length; i++) {
            maxProfit = Math.max(maxProfit, prices[i] - minPriceSoFar);
            minPriceSoFar = Math.min(minPriceSoFar, prices[i]);
        }
        return maxProfit;
    }
}
```

### その他気になったコメントなど

- scanl という考え方: https://github.com/goto-untrapped/Arai60/pull/58/files#r1782742318

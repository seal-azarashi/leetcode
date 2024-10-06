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

買い日と売り日それぞれ2つのポインタを使って管理し、各イテレーションで次の処理を行う:

- 利益が出る限りは売り日のポインタを進める
- 損益が出たら買い日のポインタを売り日のポインタに移し、その翌日を売り日とする
- 各イテレーションで損益が出ない限り最大利益の確認をする

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

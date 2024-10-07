# 122. Best Time to Buy and Sell Stock II

LeetCode URL: https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

いつ買っても売ってもいいし、期間内の値動きが全て分かってるなら、明日値が上がるなら買っておいて次の日売る、下がるなら買わない、を繰り返せば最大利益が出せるなと考え、そのように実装した。  
(💭 現実でもこれが出来たら良いのに)  

```java
/**
 * 時間計算量: O(n): 配列の全ての要素を走査し、利益が出せる場合に最大利益に加算していく
 * 空間計算量: O(1): 最大利益とイテレーションごとに算出した利益を格納する変数
 */
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null) {
            return 0;
        }

        int maxProfit = 0;
        for (int i = 0; i < prices.length - 1; i++) {
            int profit = prices[i + 1] - prices[i];
            if (profit > 0) {
                maxProfit += profit;
            }
        }
        return maxProfit;
    }
}
```

## Step 2

株を保有している、していない状態それぞれの最大利益を算出していく方法。自分では思いつかなかった。

```java
/**
 * 時間計算量: O(n): 配列の全ての要素を走査する
 * 空間計算量: O(1): 株保有、非保有時それぞれの最大利益を保持する変数
 */
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length == 0) {
            return 0;
        }

        int holdingStockProfit = -prices[0];
        int withoutStockProfit = 0;
        for (int i = 1; i < prices.length; i++) {
            holdingStockProfit = Math.max(holdingStockProfit, withoutStockProfit - prices[i]);
            withoutStockProfit = Math.max(withoutStockProfit, holdingStockProfit + prices[i]);
        }
        return withoutStockProfit;
    }
}
```

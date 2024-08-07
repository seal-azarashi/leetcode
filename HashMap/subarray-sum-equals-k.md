# 560. Subarray Sum Equals K

LeetCode URL: https://leetcode.com/problems/subarray-sum-equals-k/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

良い解法が思いつかなかったので、brute force な処理をまず実装しました。残念ながら Time Limit Exceeded。

```java
// 解いた時間: 14分ぐらい
// 時間計算量: O(n^3)
// 空間計算量: O(1)
class Solution {
    public int subarraySum(int[] nums, int k) {
        int answer = 0;
        int increasingRange = 0;
        while (increasingRange < nums.length) {
            for (int i = 0; i < nums.length - increasingRange; i++) {
                int total = 0;
                for (int j = 0; j <= increasingRange; j++) {
                    total += nums[i + j];
                }
                if (total == k) {
                    answer++;
                }
            }

            increasingRange++;
        }

        return answer;
    }
}
```

以下のようなことを考えて実装していました:

- 対象となる範囲の subarray の合計値を全て出したい
- 対象となる範囲は 1 から nums.length までの全て
- まずは範囲を1にして、次は 2、次は 3... と nums.length の合計値を出すまでイテレーションを繰り返したい
- 各イテレーションで out of bounds error を出さないように for 文の条件節を設定
- 素直に書いていたら動いたが、Time Limit Exceeded

次に解法を調べ、最もメジャーそうなものを理解した上で、ひとまず次のように実装しました。

```java
// 時間計算量: O(n)
// 空間計算量: O(n) 
class Solution {
    public int subarraySum(int[] nums, int k) {
        int count = 0;

        Map<Integer, Integer> cumulativeSumFrequency = new HashMap<>();
        cumulativeSumFrequency.put(0, 1);
        int cumulativeSum = 0;
        for (int num : nums) {
            cumulativeSum += num;
            if (cumulativeSumFrequency.containsKey(cumulativeSum - k)) {
                count += cumulativeSumFrequency.get(cumulativeSum - k);
            }
            cumulativeSumFrequency.put(cumulativeSum, cumulativeSumFrequency.getOrDefault(cumulativeSum, 0) + 1);
        }

        return count;
    }
}
```

こちらについては以下のようなことを考えて実装していました:

- ある index に対し、{そこまでの累積和 - k} と等しい累積和が以前に出現していれば、それが出現して以降の累積和が k となるという発想がエレガントで素晴らしいと思った
- 返り値を格納する変数の命名については、正確に役割を表そうとするととても長くなり、また一部削るなり単語を変えるなりしてちょうどいい長さで正確な名前をつけようと試みるもいいアイデアが思いつかず、シンプルに count とした
- 累積和の出現回数を格納する map はそのまま cumulativeSumFrequency, 累積和を格納する整数値は cumulativeSum とした
    - 最近 Go のコードをよく読んでいたので冗長に感じる気持ちがないでもないが、まあ Java なのでこれにした
- あとはなるべく素直に実装したつもり
    - 61-63 行は `count += cumulativeSumFrequency.getOrDefault(cumulativeSum - k, 0);` とも書けるが、これだとこの関数の処理の詳細に加えて getOrDefault() の挙動の両方をちゃんと把握していないと内容の理解が出来ないので、若干認知負荷が高い気がしたので今回はこのようにしてみた
        - containsKey() と get() はメソッド名だけで何してるか大体理解できるはずと考えているけど、getOrDefault() はリファレンス読まないと理解した気にはなれないと考えている
    - まあでも次の行で結局 getOrDefault() が出てくるので、先んじて登場させても別にいいかもしれない

## Step 2

## Step 3

## Step 4

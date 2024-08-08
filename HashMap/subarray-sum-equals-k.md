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
- while 文でなく for 文でいいのでは、i, j よりも適切な命名があるのでは、とリファクタリング案は頭の中にあったが、メジャーな解法をまずは把握したいので一旦終わり

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

### 累積和を使った解法

Step 1 の累積和を使った方の解答に、他の方のプルリクエストを参考にしていくつか修正を加えました。

```java
// 時間計算量: O(n)
// 空間計算量: O(n) 
class Solution {
    public int subarraySum(int[] nums, int k) {
        // 再ハッシュが起こらないように想定し得る最大 capacity を宣言時に指定
        Map<Integer, Integer> cumulativeSumFrequency = new HashMap<>(nums.length + 1);
        cumulativeSumFrequency.put(0, 1);

        int cumulativeSum = 0;
        int count = 0;
        for (int num : nums) {
            cumulativeSum += num;

            // sum(nums[0:j]) - sum(nums[0:i]) = k となる i (0 <= i < j) が x 個存在すれば
            // k と等しい部分配列 sum(nums[i:j]) も x 個存在する
            count += cumulativeSumFrequency.getOrDefault(cumulativeSum - k, 0);

            cumulativeSumFrequency.put(
                cumulativeSum,
                cumulativeSumFrequency.getOrDefault(cumulativeSum, 0) + 1
            );
        }
        return count;
    }
}
```

- 変数宣言の順序や空白行の挿入箇所を若干修正しました
- [Iwasa さんの実装](https://github.com/Yoshiki-Iwasa/Arai60/pull/15/files)を参考に再ハッシュが起こらないように想定し得る最大 capacity (nums の要素数 + 空の部分配列の和を表す1要素) を宣言時に指定するようにしました
    - 十分なメモリ容量があることを前提にしていますが、そうでない場合には考慮してほしいので、この指定が目に留まりやすいようコメントを付けています
    - Load factor も指定できるので何か性能向上に活用出来ないかと考えましたが、initial capacity の指定により再ハッシュが発生しなくなるので、活用の余地はなかったです
- fhiyo さんが以前書いたアルゴリズムを解説するコメントが素晴らしかったので拝借しました: https://github.com/fhiyo/leetcode/pull/19#discussion_r1708148290
- Step 1 で葛藤していた count のアップデート処理の書き方ですが、結局 getOrDefault() を使って一行で記述するように修正しました
    - containsKey() と get() を代わりに使うのが読みやすさに寄与しない気がしたのが理由です (自分が Java のライブラリ活用に慣れてしまっているだけかもしれませんが...)
- put() の呼び出しをしている行が長かったので改行を入れました (実際の開発では formatter がやってくれるので、あくまで今見やすくするためにやっているんですよ、というのは面接の場では伝えたいところです)

### TODO

- ひとつめの計算量って本当に正しい？げんみつにはいくら？参考: https://github.com/goto-untrapped/Arai60/pull/28/files#r1641923067
- 累積和の理解度を深めてくれそうな oda さんのメモ: https://github.com/kazukiii/leetcode/pull/17/files#r1649125989
- やっぱ brute force な書き方も復習するか
    - 三重ループじゃなくて二重ループにできるらしい: https://github.com/Mike0121/LeetCode/pull/33/files#diff-b46377e322184ae9a85cc56e120560a4f00c8d761c58a4b284f09bfec7143428
    - i, j じゃなくて begin, end とかがいい: https://github.com/goto-untrapped/Arai60/pull/28#discussion_r1642960547
    - 
- この類題、累積和の理解度チェックにいいかも: https://github.com/SuperHotDogCat/coding-interview/pull/29#issuecomment-2169284348

## Step 3

## Step 4
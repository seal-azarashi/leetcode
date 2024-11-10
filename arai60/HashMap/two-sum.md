# 1. Two Sum

LeetCode URL: https://leetcode.com/problems/two-sum/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 10 m 46 s
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public int[] twoSum(int[] nums, int target) {
        // Key: target - num, Value: index of num
        HashMap<Integer, Integer> addedUpTo = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            if (addedUpTo.containsKey(nums[i])) {
                return new int[]{addedUpTo.get(nums[i]), i};
            }

            addedUpTo.put(target - nums[i], i);
        }

        return new int[2];
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 昔解いたことがあり、ベストな計算量の実装方法を知っていたのでそのまま書いた
- 多分、面接で出題されたらいきなり答えを言わず、まずは時間計算量 O(n^2) の brute force アプローチから始めて、その改善案としてこの解法の説明をしたいと思った
    - brute force アプローチも空間計算量においては優れるので、(より優れた時間計算量を差し置いてでも) そのアドバンテージを活かしたいというニーズが無いか、というのも、出題形式によっては話せるとよさそうに思う
- HashMap の名前をどうするか悩んだ
    - map では何を使うのかの情報がない
    - key, value の中身やその意図を名前にまるごと含めると長くなりすぎる
    - いちばん重要なのは、key が「足すと target になる数」であることなので、addedUpTo としたうえで、コードコメントを添えることにした

## Step 2

Step 1 の実装を少し修正しました。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        // Key: target - num, Value: index of num
        HashMap<Integer, Integer> complementMap = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (complementMap.containsKey(nums[i])) {
                return new int[]{complementMap.get(nums[i]), i};
            }
            complementMap.put(target - nums[i], i);
        }

        throw new IllegalArgumentException("No two sum solution.");
    }
}
```

以下思考しながら修正しています:

- HashMap の key になる値を complement と表現している記事がいくつかあったので、上記 HashMap を `complementMap` に修正した
    - 一例:
        - https://medium.com/swlh/two-ways-to-solve-the-two-sum-problem-a158d593eef
        - https://dev.to/shavonharrisdev/two-sum-problem-a-straightforward-guide-209l
    - Wikipedia も一応眺めたけど問題なさそう: https://ja.wikipedia.org/wiki/%E8%A3%9C%E6%95%B0
- constraints に "Only one valid answer exists" とあるので、最後の行には到達しないはずだ、ということを示すために `IllegalArgumentException` を throw するようにした
    - 実際仕事で書くのは不親切過ぎるよね、これはあくまで Leetcode の出題形式に合わせてるだけだからね、というのは、面接の際ちゃんと申し添えるようにしたい
    - 結構議論の余地のある修正な気もするので、修正前にインタビュワーと合意を取りたいかも

以下、他の人のプルリクエストを読んで感じたことを列挙しています:

- [oda さんの過去のレビューコメント](https://github.com/Yoshiki-Iwasa/Arai60/pull/10#discussion_r1647158610)を読んで、なるほどこういった考えかたをすれば、答えを知らずとも自然に正解にたどり着きそう
- Step 1 で出てこなかった他の解法を把握できて良かった:
    - ソートして対応する要素を探すために二分探索: TC: O(n log n), SC: O(n log n)
    - ソートして左端、右端に設置した 2 pointers を徐々に近づけていく: TC: O(n log n), SC: O(n)
- 両方 Step 1 の段階で思い浮かべられるように思考していきたい

## Step 3

Submit したコードは次のようになりました。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> complementMap = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (complementMap.containsKey(nums[i])) {
                return new int[] {complementMap.get(nums[i]), i};
            }
            complementMap.put(target - nums[i], i);
        }

        throw new IllegalArgumentException("No two sum pair found.");
    }
}
```

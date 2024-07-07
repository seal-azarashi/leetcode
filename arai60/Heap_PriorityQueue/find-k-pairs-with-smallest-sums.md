# 373. Find K Pairs with Smallest Sums

LeetCode URL: https://leetcode.com/problems/find-k-pairs-with-smallest-sums/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 解いた時間: 40分ぐらい
// 時間計算量: O(k log k)
// 空間計算量: O(k)
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        PriorityQueue<int[]> indiceOfKSmallestPairs = new PriorityQueue<>((a, b) -> (nums1[a[0]] + nums2[a[1]]) - (nums1[b[0]] + nums2[b[1]]));
        for (int i = 0; i < nums1.length; i++) {
            indiceOfKSmallestPairs.offer(new int[]{i, 0});
        }

        List<List<Integer>> answer = new ArrayList<>();
        for (int i = 0; i < k; i++) {
            int[] pair = indiceOfKSmallestPairs.poll();
            int nums1Index = pair[0];
            int nums2Index = pair[1];
            answer.add(Arrays.asList(nums1[nums1Index], nums2[nums2Index]));
            
            if (nums2Index + 1 < nums2.length) {
                indiceOfKSmallestPairs.offer(new int[]{nums1Index, nums2Index + 1});
            }
        }

        return answer;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 最初に次の解法を思いつく: すべての組み合わせを PriorityQueue (Min Heap) に入れ、合計値が少ない上位 k のペアを返す
- これだと計算量が非常に大きいので別の方法を模索するが、期待通りに機能するものが見つからない
- 見つからないので最初に思いついた解法を試しに動かしてみるも案の定 Time Limit Exceeded
- 解法を確認し、10分ほどで理解して実装した

## Step 2

```java
// 時間計算量: O(k log k)
// 空間計算量: O(k)
class Solution {
    private record Pair(int nums1Index, int nums2Index) {};

    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        PriorityQueue<Pair> kSmallestSumPairs = new PriorityQueue<>(
            (a, b) -> (nums1[a.nums1Index] + nums2[a.nums2Index]) - (nums1[b.nums1Index] + nums2[b.nums2Index])
        );
        for (int i = 0; i < nums1.length && i < k; i++) {
            kSmallestSumPairs.offer(new Pair(i, 0));
        }

        List<List<Integer>> answer = new ArrayList<>(k);
        for (int i = 0; i < k; i++) {
            Pair pair = kSmallestSumPairs.poll();
            answer.add(List.of(nums1[pair.nums1Index], nums2[pair.nums2Index]));
            
            if (pair.nums2Index + 1 < nums2.length) {
                kSmallestSumPairs.offer(new Pair(pair.nums1Index, pair.nums2Index + 1));
            }
        }

        return answer;
    }
}
```

- 基底ロジックはそのままで、Step 1 で実施できていなかったいくつかの処理効率化を実施
    - PriorityQueue に k 個以上の要素はいらないので、最初の for 文の条件式を修正
    - answer に対していたずらにメモリ再割り当てが行われないよう宣言時にサイズを指定
    - answer の要素はミュータブルだったり null を許容をする必要がないので、Arrays.asList の代わりに List.of を使用 (副作用で 4ms ほど早くなった)
- 要素数2の配列だと意図が汲み取りづらいため、インデックスのペアを record Pair に格納
- PriorityQueue の宣言行が長くなったので改行 (単純に見づらくなってしまったので好みで実施したが、仕事でやるならフォーマッターやチームのコーディングルールに従いたいところ)
- 他の方の解答を見ると以下のようなアプローチのものもあったが、実装してみると Leetcode のジャッジ上では上記の実装が最も高速かつメモリ使用量も悪くなかったので、ロジックはそのままにした:
    - PriorityQueue に対して、一度のイテレーションで最大2つの要素を追加する
    - 走査済のペアを hash set に保存し、重複して処理が行われないようにする
- とはいえ多分面接で思いつくとしたらこっちなんだろうなとは思う

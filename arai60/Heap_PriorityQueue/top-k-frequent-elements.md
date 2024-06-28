# 347. Top K Frequent Elements

LeetCode URL: https://leetcode.com/problems/top-k-frequent-elements/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 20 m 36 s
// 時間計算量: O(n log n), 空間計算量: O(n)
class Solution {
    private record NumFrequency(int num, int frequency) {};

    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> numFrequencyMap = new HashMap<>();
        for (int num : nums) {
            numFrequencyMap.put(
                num,
                numFrequencyMap.getOrDefault(num, 0) + 1
            );
        }

        PriorityQueue<NumFrequency> frequentNums = new PriorityQueue<>((a, b) -> b.frequency - a.frequency);
        for (int num : numFrequencyMap.keySet()) {
            frequentNums.offer(
                new NumFrequency(
                    num,
                    numFrequencyMap.get(num)
                )
            );
        }

        int[] answer = new int[k];
        for (int i = 0; i < k; i++) {
            answer[i] = frequentNums.poll().num;
        }
        return answer;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 最初に次の解法を思いつく:
    1. HashMap に nums 内のユニークな数とその登場回数を入れる
    2. 登場回数の降順で整列させる PriorityQueue に上記 HashMap の要素を入れる
    3. 上記 PriorityQueue から k 個値を取り出して返す
- 計算量は次の通りと考え、悪くないので実装をしようと決める:
    - 時間計算量は O(n log n):
        - HashMap に要素を入れるのに n 回 (n = nums.length)
        - PriorityQueue に要素を入れるのに m 回 (m = nums 内のユニークな要素数) 
        - PriorityQueue から要素を取り出すのに k 回
        - 上記合わせて n + m log m + k
        - n, m, k が同じ数になる最悪ケースを考慮し、big O notation での記述は O(n log n)
    - 空間計算量は O(n)
- そういえば最近の Java には Record があるのを思い出し、ググって確認しながら書く
- HashMap のメソッドの記憶が怪しかったので、公式ドキュメントを見て put, getOrDefault を使えばいいと確認
- PriorityQueue のメソッドの記憶が怪しかったので、公式ドキュメントを見て get, offer を使えばいいと確認
    - offer と同じ処理をする add もあるが、Queue 由来の offer を用いる方が PriorityQueue を使う上で適当だと思う
- 計20分ほどで完了

## Step 2

バケットソート、クイックセレクトで実装してみたい
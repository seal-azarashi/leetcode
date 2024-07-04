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
            numFrequencyMap.put(num, numFrequencyMap.getOrDefault(num, 0) + 1);
        }

        PriorityQueue<NumFrequency> frequentNums = new PriorityQueue<>((a, b) -> b.frequency - a.frequency);
        for (int num : numFrequencyMap.keySet()) {
            frequentNums.offer(new NumFrequency(num, numFrequencyMap.get(num)));
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

2つ新しい解法を追加しました。レビュワーの方は Step 1 の方も確認お願いします🙏

### バケットソート

[kagetora0924 さんの解答](https://github.com/kagetora0924/leetcode-grind/blob/5946807f887a54530df82f471d6fd422e8ba875f/Arai60/Heap_Priority_Queue/347/347_1-3.py)に時間計算量 O(n) のバケットソートを使ったものがあったので実装してみました。
講師のお二人の反応見る限りそんなに重要そうでもなさそうだけど一応: https://github.com/kazukiii/leetcode/pull/10#discussion_r1639979474

```java
// 時間計算量: O(n), 空間計算量: O(n)
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> numFrequencyMap = new HashMap<>();
        for (int num : nums) {
            numFrequencyMap.put(
                num,
                numFrequencyMap.getOrDefault(num, 0) + 1
            );
        }

        List<Integer>[] buckets = new List[nums.length + 1];
        for (int num : numFrequencyMap.keySet()) {
            int frequency = numFrequencyMap.get(num);
            if (buckets[frequency] == null) {
                buckets[frequency] = new ArrayList<>();
            }
            buckets[frequency].add(num);
        }

        int[] answer = new int[k];
        int answerIndex = 0;
        for (int i = buckets.length - 1; i >= 0; i--) {
            if (buckets[i] == null) {
                continue;
            }

            for (int num : buckets[i]) {
                if (answerIndex >= k) {
                    break;
                }

                answer[answerIndex] = num;
                answerIndex++;
            }
        }
        return answer;
    }
}
```

時間計算量は優れていますが、空間計算量は固定で nums.length になってしまいますね。ステップ1の方の解法だと空間計算量はユニークな要素数でした。

### クイックセレクト

クイックセレクトの理解度チェックも兼ねて書いてみる。

```java
// 時間計算量:
//     - avg: O(n log n)
//     - worst: O(n^2)
// 空間計算量: O(n)
class Solution {
    private record NumFrequency(int num, int frequency) {};

    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> numFrequencyMap = new HashMap<>();
        for (int num : nums) {
            numFrequencyMap.put(num, numFrequencyMap.getOrDefault(num, 0) + 1);
        }

        NumFrequency[] numFrequencyArray = new NumFrequency[numFrequencyMap.size()];
        int numFrequencyArrayIndex = 0;
        for (int num : numFrequencyMap.keySet()) {
            numFrequencyArray[numFrequencyArrayIndex] = new NumFrequency(num, numFrequencyMap.get(num));
            numFrequencyArrayIndex++;
        }
        quickSelect(numFrequencyArray, k);

        int[] answer = new int[k];
        for (int i = 0; i < k; i++) {
            answer[i] = numFrequencyArray[i].num;
        }
        return answer;
    }

    private void quickSelect(NumFrequency[] numFrequencyArray, int k) {
        int left = 0;
        int right = numFrequencyArray.length - 1;
        while (left < right) {
            int pivot = partition(numFrequencyArray, left, right);
            if (pivot == k - 1) {
                return;
            } else if (pivot < k - 1) {
                left = pivot + 1;
            } else {
                right = pivot - 1;
            }
        }
    }

    private int partition(NumFrequency[] numFrequencyArray, int left, int right) {
        int pivotFrequency = numFrequencyArray[right].frequency;
        int leftPartitionBoundary = left - 1;
        for (int i = left; i < right; i++) {
            if (numFrequencyArray[i].frequency >= pivotFrequency) {
                leftPartitionBoundary++;
                swapArrayElements(numFrequencyArray, leftPartitionBoundary, i);
            }
        }

        int pivot = leftPartitionBoundary + 1;
        swapArrayElements(numFrequencyArray, pivot, right);
        return pivot;
    }

    private void swapArrayElements(NumFrequency[] numFrequencyArray, int left, int right) {
        NumFrequency temp = numFrequencyArray[left];
        numFrequencyArray[left] = numFrequencyArray[right];
        numFrequencyArray[right] = temp;
    }
}
```

## Step 3

TODO

## Step 4

[Yoshiki-Iwasa さんのレビュー](https://github.com/seal-azarashi/leetcode/pull/9/files#r1665393659)を受け、PriorityQueue に入る要素数が k を超えないように変えました (厳密に言うと for 文の中で一時的に k + 1 になる可能性はあります)。  
加えて、走査対象の frequency が PriorityQueue 内最小の要素のそれよりも小さい場合は PriorityQueue への挿入・削除は行わないようにしました。

```java
class Solution {
    private record NumFrequency(int num, int frequency) {};

    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> numFrequencyMap = new HashMap<>();
        for (int num : nums) {
            numFrequencyMap.put(num, numFrequencyMap.getOrDefault(num, 0) + 1);
        }

        PriorityQueue<NumFrequency> frequentNums = new PriorityQueue<>((a, b) -> a.frequency - b.frequency);
        for (int num : numFrequencyMap.keySet()) {
            if (frequentNums.size() == k && numFrequencyMap.get(num) <= frequentNums.peek().frequency) {
                continue;
            }

            frequentNums.offer(new NumFrequency(num, numFrequencyMap.get(num)));
            if (frequentNums.size() > k) {
                frequentNums.poll();
            }
        }

        int[] answer = new int[k];
        for (int i = 0; i < k; i++) {
            answer[i] = frequentNums.poll().num;
        }
        return answer;
    }
}
```

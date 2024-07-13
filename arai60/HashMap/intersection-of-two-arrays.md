# 349. Intersection of Two Arrays

LeetCode URL: https://leetcode.com/problems/intersection-of-two-arrays/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 解いた時間: 13分ぐらい
// 時間計算量: O(n + m)
// 空間計算量: O(n) 
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> num1UniqueValues = new HashSet<>();
        for (int num : nums1) {
            num1UniqueValues.add(num);
        }

        Set<Integer> intersection = new HashSet<>();
        for (int num : nums2) {
            if (num1UniqueValues.contains(num)) {
                intersection.add(num);
            }
        }

        int[] result = new int[intersection.size()];
        int resultIndex = 0;
        for (int num : intersection) {
            result[resultIndex++] = num;
        }
        return result;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 片方の配列にあるものを保持しておき、もう片方の配列の操作時にその set にある値だけ保存して返す解法を思いつく
- 保持する要素はユニークでなければはらないので HashSet を使いたい
- HashSet の要素を配列に変換する、ないし走査するメソッドが思い浮かばなかったので公式ドキュメントを見るも、都合がよさそうなものがないので for-each で走査することを決める
- HashSet 一つで実装出来そうな気がするも、時間内にいい案は思いつかなさそうだったので2つの HashSet で実装する
- 想定通りに動いて submit した

## Step 2

Step 1 のコードを少し改良しました。

```java
// 時間計算量: O(n + m)
// 空間計算量: O(min(n, m))
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        if (nums1.length > nums2.length) {
            return intersection(nums2, nums1);
        }

        Set<Integer> num1UniqueValues = new HashSet<>();
        for (int num : nums1) {
            num1UniqueValues.add(num);
        }

        Set<Integer> intersection = new HashSet<>();
        for (int num : nums2) {
            if (num1UniqueValues.contains(num)) {
                intersection.add(num);
            }
        }

        int[] result = new int[intersection.size()];
        int resultIndex = 0;
        for (int num : intersection) {
            result[resultIndex++] = num;
        }
        return result;
    }
}
```


- set を一つにする方法として新たにリストを導入することを思いついたが、空間計算量の節約にはならない (むしろインデックス等のデータ量が増えるのでマイナスになる見込み) ので採用を見送った
- 空間計算量を書き出してみたら O(n + 2 * min(n, m)) で、n を min(n, m) にする余地があると気付いき、最初の分岐を追加した
    - 両配列の要素数の差がほとんどなければスタックフレームが一つ増えた分よりメモリ空間が圧迫されてしまう懸念はあるものの、スタックフレーム一つの増加がクリティカルな影響を与えることは基本ないはずなので採用
    - リソースが極限まで限られた状況だと別のアプローチを考えた方がいいかも

他の人の実装を読んでいて思ったことも書きます

- [sort を用いる案](https://github.com/SuperHotDogCat/coding-interview/blob/4822aea69850ace467725b4d47865e1e184b35f0/arai60/intersection_of_the_two_arrays/phase2.py)が、時間計算量は多くなるものの、空間計算量に関してはひとつ set の数が減るので利点があるように見えたが、ソート (TimSort) 実行時も最大で O(n) のメモリ空間が必要となるので、空間計算量においても特に利点はないと判断した。
- [Python だと非常に簡潔に書ける](https://github.com/SuperHotDogCat/coding-interview/blob/4822aea69850ace467725b4d47865e1e184b35f0/arai60/intersection_of_the_two_arrays/phase3.py)のを見て、やっぱ Python で対策するの良いなって思った
- [Rust の標準ライブラリを用いた実装](https://github.com/Yoshiki-Iwasa/Arai60/blob/b370c6059807346b0ae6462629609b8d33d3e0bf/problems/src/intersection_of_two_arrays/step4.rs)を見て、Java の Stream API で似たような実装にできるなと思いつつ、デバッグのしづらさやオーバーヘッドが増加する可能性があることを考えると、コーディング面接の解答に使うのは適当ではないかなと思った。
- [Set を使わずに merge sort っぽく書く方法](https://github.com/TORUS0818/leetcode/pull/15/files#diff-b91a580511b41f0315a57fe663d646b2b12fdabb7b4c4c3495ed069d8d6974fdR126-R157)をいろんな人が書いていて気になった
    - TODO: merge sort を復習してから自分でもやってみよう

## Step 3

大体3~4分で書けた。

```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        // 空間計算量削減のため、要素数が少ない方の配列を第一引数にして処理する
        if (nums1.length > nums2.length) {
            return intersection(nums2, nums1);
        }

        Set<Integer> nums1Set = new HashSet<>();
        for (int num : nums1) {
            nums1Set.add(num);
        }

        Set<Integer> intersection = new HashSet<>();
        for (int num : nums2) {
            if (nums1Set.contains(num)) {
                intersection.add(num);
            }
        }

        int[] result = new int[intersection.size()];
        int resultIndex = 0;
        for (int num : intersection) {
            result[resultIndex++] = num;
        }
        return result;
    }
}
```

- uniqueValues は set と言い換えて差し支えないと判断して set の変数名を num1Set に変奥
- 最初の if 文がなんのためにあるのかは、上から順にまず読むだけだと理解しづらいかもと思いコメントを追加

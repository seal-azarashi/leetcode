# 153. Find Minimum in Rotated Sorted Array

LeetCode URL: https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

答えは知っていたのですぐ書けたが、充分に理解出来ているのかと言われると自信がないので、頭の中の整理を兼ねて何をしているのか書き出してみる。

- 2つのポインタを保持しながら二分探索を行い、最小の値がある位置を特定する
    - left: これが指す位置よりも左には最小の値がないことを示す
    - right: これが指す位置よりも右には最小の値がないことを示す
- 各イテレーションで探索範囲の中央にある値を確認し、
    - 次のどちらかの処理を行う:
        - right の値と同じかそれよりも小さい場合、right を中央の値の位置に移動させる
        - 上の条件に当てはまらない場合、中央の値の位置のひとつ右の位置に left を移動させる
    - right を最小の値がある位置に向かわせ、left には最大位置と最小位置の境目を探させるイメージ

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] <= nums[right]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return nums[right];
    }
}
```

left, right は配列内での位置関係に合わせて命名しているが、もっと具体的な役割を表すような名前にしてもいいかもと思った

- しかしいい名前が思いつかない
    - left: rotationBoundary
    - right: minValueExplorer
- 最初に変数名だけ見てもわからなさそうなので、やはり left, right ぐらいにしておくのがいいか

## Step 2

### 最小の値を左端に持つ範囲を探索するアプローチ

最小の値そのものでなく、それを左端に持つ範囲を探索する。

各イテレーションでは left を最大位置と最小位置の境目に移動させることを目的とした処理が行われる。right はその際行われる二分探索処理に必要。

While 文の中で範囲が見つかったかどうか判定させた方がコードの見通しが良くなると考え、引数が valid であればこれの外の範囲には出ないことを前提に実装した。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            if (nums[left] <= nums[right]) {
                return nums[left];
            }

            int middle = left + (right - left) / 2;
            if (nums[left] <= nums[middle]) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }
        throw new IllegalArgumentException("Unreachable if the argument is valid");
    }
}
```

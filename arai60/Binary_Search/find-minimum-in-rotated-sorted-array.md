# 153. Find Minimum in Rotated Sorted Array

LeetCode URL: https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

答えを知っていたので書けはしたのですが、手癖ではなく充分な理解を持った上で解けているか自信がないので、この解法に対する理解を以下に書き出します。フィードバックいただけますと嬉しいです。

- 閉区間で二分探索を行い、最小の値から始まる範囲とそうでない範囲の境目を探す
    - 例えば `[3,4,5,1,2]` の場合、最小の値から始まる範囲に属する値を T 、そうでないものを F とすると `[F, F, F, T, T]` となるが、この T の中で一番左側にあるものを探していくイメージ
- 各イテレーションで探索範囲の中央にある値 (以下 middle) を確認し、次のどちらかの処理を行う:
    - middle の位置にある値が right の位置にある値よりも小さい場合、最小の値はそこかそれよりも左の位置にあるため、right を middle の位置に移動させる
    - 上の条件に当てはまらない場合、最小の値はそれよりも右の位置にあるため、middle のひとつ右の位置に left を移動させる
- 各ポインタは以下示しているため、同じ位置を指したら即ちその位置に存在する値が最小の値であると判断する:
    - left: これが指す位置よりも左には最小の値が存在しないことを示す
    - right: これが指す位置にある値が、最小の値かそれよりも右に存在するものであることを示す

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
        return nums[left];
    }
}
```

## Step 2

### 右端の値を用いて比較を行うパターン

何人か他の方が書かれていたので書いてみました。  
右端の値が、最小の値かそれよりも右にあるいずれかの値であることを利用した条件判定により二分探索を行います。

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] <= nums[nums.length - 1]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return nums[left];
    }
}
```

### While 文終了前に return させるパターン

Step 1 の解法の処理効率向上のため、最小の値が存在する位置が見つかったらその時点で return させるよう修正します。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            if (nums[left] < nums[right]) {
                return nums[left];
            }

            int middle = left + (right - left) / 2;
            if (nums[middle] <= nums[right]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return nums[left];
    }
}
```

## Step 3

処理効率の良さでこちらを選びました。  
好みかもしれませんが、右端の値を用いて比較を行うパターンよりも直感的にも思えます。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int findMin(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left < right) {
            if (nums[left] < nums[right]) {
                return nums[left];
            }

            int middle = left + (right - left) / 2;
            if (nums[middle] <= nums[right]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return nums[left];
    }
}
```

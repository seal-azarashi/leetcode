# 35. Search Insert Position

LeetCode URL: https://leetcode.com/problems/search-insert-position/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

Binary search を実施して値が見つからなかった場合、while 文を抜けた時点で left ポインタが探索対象の値の次に大きい値を指すことを理解し、次のように書いた。好みで閉区間にしている。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            }
            if (nums[middle] < target) {
                left = middle + 1;
            }
            if (target < nums[middle]) {
                right = middle - 1;
            }
        }
        return left;
    }
}
```

## Step 2

### 半開区間 (左閉右開) の実装

半開区間でも書いてみる。こちらの方が一般的な印象があるので以後はこちらをベースに書いていく。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length;
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            }
            if (nums[middle] < target) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }
        return left;
    }
}
```

### 必ず while 文を抜ける実装

途中で処理を中断せず while 文を抜けた場合、その時点で left ポインタは探索対象、もしくはその次に大きい値を指している。これを踏まえて step 1 の処理を、必ず while 文を抜けるように修正してみる。  

"return the index if the target is found" とある問題文の指示を反映している感が薄まったような印象。  
一方、どの実装についても内容の理解には上記の left ポインタの特徴を把握することが避けられないのを考えると、これを把握している読み手ならまあ迷うこともないなずなのでこれでもいいのかなと思った。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length;
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] < target) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }
        return left;
    }
}
```

## Step 3

必ず while 文を抜ける実装を採用。

```java
/**
 * 解いた時間: 2分未満
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left = 0, right = nums.length;
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] < target) {
                left = middle + 1;
            } else {
                right = middle;
            }
        }
        return left;
    }
}
```

## Step 4

### Arrays クラスを用いる解法

ahayashi さんの指摘で Arrays.binarySearch() があったことを思い出したので加筆。

```java
/**
 * 時間計算量: O(log n)
 * 空間計算量: O(1)
 */
class Solution {
    public int searchInsert(int[] nums, int target) {
        int foundIndex = Arrays.binarySearch(nums, target);
        if (0 < foundIndex) {
            return foundIndex;
        }

        return -(foundIndex + 1);
    }
}
```

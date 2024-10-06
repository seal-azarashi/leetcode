# 198. House Robber

LeetCode URL: https://leetcode.com/problems/house-robber/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

最初に思いついた、ある地点での最大 amount of money をキャッシュしていく実装。引数に対して破壊的変更を起こさないためにキャッシュ用の配列を宣言しているが、これをなくして引数を代わりに使えば空間計算量を O(1) にすることは可能。  
Constraints では配列の要素数は1以上となっているが、一応0だったときでも対応出来るような実装にした。他にも要素が負の整数になる可能性を考慮しようかとも考えたが、泥棒が盗める財産の料がマイナスになるというのはあり得ないし、仮にそういった不正なデータが渡される可能性があるとしても、それをチェックする役割をこの関数に持たせるべきでは無いと考え、考慮はしないこととした。  

```java
/**
 * かかった時間: 約15分
 * 時間計算量: O(n): 
 *     - O(n): キャッシュ用配列の宣言 (引数に受け取る配列のコピー)
 *     - O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(n):
 *     - O(n): キャッシュ用配列
 *     - O(1): return する結果を格納する変数
 */
class Solution {
    public int rob(int[] nums) {
        int[] amountCache = Arrays.copyOfRange(nums, 0, nums.length);
        int maxAmount = 0;
        for (int i = 0; i < amountCache.length; i++) {
            if (i == 2) {
                amountCache[i] += amountCache[i - 2];
            } else if (2 < i) {
                amountCache[i] += Math.max(amountCache[i - 3], amountCache[i - 2]);
            }
            maxAmount = Math.max(maxAmount, amountCache[i]);
        }
        return maxAmount;
    }
}
```

## Step 2

### 配列を用いたキャッシュ (Step 1 のブラッシュアップ)

配列が null だったときの考慮があってもいいかと思いチェックを入れた。

```java
/**
 * 時間計算量: O(n): 
 *     - O(n): キャッシュ用配列の宣言 (引数に受け取る配列のコピー)
 *     - O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(n):
 *     - O(n): キャッシュ用配列
 *     - O(1): return する結果を格納する変数
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null) {
            return 0;
        }

        int[] amountCache = Arrays.copyOfRange(nums, 0, nums.length);
        int maxAmount = 0;
        for (int i = 0; i < amountCache.length; i++) {
            if (i == 2) {
                amountCache[i] += amountCache[i - 2];
            } else if (2 < i) {
                amountCache[i] += Math.max(amountCache[i - 3], amountCache[i - 2]);
            }
            maxAmount = Math.max(maxAmount, amountCache[i]);
        }
        return maxAmount;
    }
}
```

### 2つの変数を用いたキャッシュ

他の方の解答を参考に書いた。より空間計算量が少ない。

```java
/**
 * 時間計算量: O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(1): ひとつ前、ふたつ前の時点での盗めるお金の最大値を格納する2つの変数
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }

        int maxAmount = 0;
        int twoBackMaxAmount = 0, oneBackMaxAmount = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int currentMaxAmount = Math.max(twoBackMaxAmount + nums[i], oneBackMaxAmount);
            maxAmount = Math.max(maxAmount, currentMaxAmount);

            twoBackMaxAmount = oneBackMaxAmount;
            oneBackMaxAmount = currentMaxAmount;
        }
        return maxAmount;
    }
}
```

前の実装に引きずられて maxAmount を使っていたが、イテレーション終了時点で oneBackMaxAmount が必ず大きくなるようにキャッシュしているので、代わりにそちら返り値にする。

```java
/**
 * 時間計算量: O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(1): ひとつ前、ふたつ前の時点での盗めるお金の最大値を格納する2つの変数
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }

        int twoBackMaxAmount = 0, oneBackMaxAmount = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int maxAmount = Math.max(twoBackMaxAmount + nums[i], oneBackMaxAmount);

            twoBackMaxAmount = oneBackMaxAmount;
            oneBackMaxAmount = maxAmount;
        }
        return oneBackMaxAmount;
    }
}
```

配列の長さが1のときの考慮をしなくていいと気づいて修正。

```java
/**
 * 時間計算量: O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(1): ひとつ前、ふたつ前の時点での盗めるお金の最大値を格納する2つの変数
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int twoBackMaxCount = 0, oneBackMaxCount = 0;
        for (int i = 0; i < nums.length; i++) {
            int maxCount = Math.max(twoBackMaxCount + nums[i], oneBackMaxCount);

            twoBackMaxCount = oneBackMaxCount;
            oneBackMaxCount = maxCount;
        }
        return oneBackMaxCount;
    }
}
```

### トップダウンアプローチ

他の方の解法にあったので書いてみた。スタックオーバーフローのリスクがあるので注意。

```java
/**
 * 時間計算量: O(n): 
 *     - O(n): キャッシュ用配列の宣言と初期化
 *     - O(n): ヘルパー関数の呼び出しが最悪の場合各要素に対して一度行われる (二度目以降はキャッシュされた値が返る)
 * 空間計算量: O(n): キャッシュ用配列
 */
class Solution {
    private static final int NOT_CACHED = -1;

    private int[] amountCache;
    private int[] nums;
    
    public int rob(int[] nums) {
        if (nums == null) {
            return 0;
        }

        this.nums = nums;
        amountCache = new int[nums.length];
        Arrays.fill(amountCache, NOT_CACHED);
        return robHelper(nums.length - 1);
    }
    
    private int robHelper(int numsIndex) {
        if (numsIndex < 0) {
            return 0;
        }
        if (this.amountCache[numsIndex] != NOT_CACHED) {
            return this.amountCache[numsIndex];
        }
        
        this.amountCache[numsIndex] = Math.max(robHelper(numsIndex - 2) + this.nums[numsIndex], robHelper(numsIndex - 1));
        return this.amountCache[numsIndex];
    }
}
```

## Step 3

```java
/**
 * 時間計算量: O(n): 配列の各要素に対しての一連の定数時間で行われる処理
 * 空間計算量: O(1): ひとつ前、ふたつ前の時点での盗めるお金の最大値を格納する2つの変数
 */
class Solution {
    public int rob(int[] nums) {
        if (nums == null || nums.length == 0) {
            return 0;
        }

        int twoBackMaxCount = 0, oneBackMaxCount = 0;
        for (int i = 0; i < nums.length; i++) {
            int maxCount = Math.max(twoBackMaxCount + nums[i], oneBackMaxCount);

            twoBackMaxCount = oneBackMaxCount;
            oneBackMaxCount = maxCount;
        }
        return oneBackMaxCount;
    }
}
```

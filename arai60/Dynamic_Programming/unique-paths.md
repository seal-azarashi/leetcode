# 62. Unique Paths

LeetCode URL: https://leetcode.com/problems/unique-paths/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

DP の問題なのでなんかキャッシュとかするんだろうなとは思うが、いい案が思いつかない。しかしながら[前回の教訓](https://github.com/seal-azarashi/leetcode/pull/30/files#diff-f8fa021ba9e65116bf33b3b3f8db9c427a3326861bb34e4dabf63348823cee80R128-R151)を踏まえ、いきなり解答は見ずに、思いついた解法書いてみてブラッシュアップしていくことを試みる。  
まず思いついたのは愚直にスタートからゴールまでの道順を網羅的に走査していくパターン。 TLE。

```java
/**
 * 時間計算量: O(2^(m + n)): 
 *     - 各マスで進行方向の候補が最大で2つある (端にいる場合は1つ)
 *     - スタートからゴールに到達するまで m - 1 + n - 1 回の移動が必要
 * 空間計算量: O(m + n):
 *     - スタートからゴールまでの移動回数が再帰の深さになる
 *     - 移動回数は m - 1 + n - 1 回
 */
class Solution {
    int uniquePathCount, m, n;

    public int uniquePaths(int m, int n) {
        this.uniquePathCount = 0;
        this.m = m;
        this.n = n;
        findUniquePathRecursively(0, 0);
        return this.uniquePathCount;
    }

    private void findUniquePathRecursively(int x, int y) {
        if (x == n - 1 && y == m - 1) {
            this.uniquePathCount++;
            return;
        }
        if (x < this.n) {
            findUniquePathRecursively(x + 1, y);
        }
        if (y < this.m) {
            findUniquePathRecursively(x, y + 1);
        }
    }
}
```

同じアプローチでゴールからも走査が出来るなと考え処理の開始地点を変えてみる。 TLE。

```java
/**
 * 時間計算量: O(2^(m + n)): 
 *     - 各マスで進行方向の候補が最大で2つある (端にいる場合は1つ)
 *     - スタートからゴールに到達するまで m - 1 + n - 1 回の移動が必要
 * 空間計算量: O(m + n):
 *     - スタートからゴールまでの移動回数が再帰の深さになる
 *     - 移動回数は m - 1 + n - 1 回
 */
class Solution {
    int uniquePathCount, m, n;

    public int uniquePaths(int m, int n) {
        this.uniquePathCount = 0;
        this.m = m;
        this.n = n;
        findUniquePathRecursively(n - 1, m - 1);
        return this.uniquePathCount;
    }

    private void findUniquePathRecursively(int x, int y) {
        if (x == 0 && y == 0) {
            this.uniquePathCount++;
            return;
        }
        if (0 <= x) {
            findUniquePathRecursively(x - 1, y);
        }
        if (0 <= y) {
            findUniquePathRecursively(x, y - 1);
        }
    }
}
```

ゴールからの走査なら、移動対象になる2つの走査済のマスからの unique path の合算値がそのマスからの unique path になると気づき、 memoization のための二次元配列を使うようにした。テストケースを全てパス。  
エッジケースの考慮に苦労した。 Leetcode のテストケースが用意されていないとここまで作り込めなかった気がする... 採点ツールを使わない、ホワイトボードに書くような形式の面接だと不合格となってしまうだろうか？

```java
/**
 * 時間計算量: O(m * n):
 *     - O(m * n): 配列の宣言
 *     - O(m + n): 配列の底と右端の要素を 1 にする
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでのマスの位置の判定や値の計算、代入といった処理
 * 空間計算量: O(m * n): キャッシュ用の二次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        if (m == 1 && n == 1) {
            return 1;
        }

        int[][] pathCountCache = new int[m][n];
        for (int y = m - 1; y >= 0; y--) {
            for (int x = n - 1; x >= 0; x--) {
                boolean inXBorder = x + 1 == n, inYBorder = y + 1 == m;
                if (inXBorder && inYBorder) {
                    pathCountCache[y][x] = 0;
                } else if (inXBorder || inYBorder) {
                    pathCountCache[y][x] = 1;
                } else {
                    pathCountCache[y][x] = pathCountCache[y][x + 1] + pathCountCache[y + 1][x];
                }
            }
        }
        return pathCountCache[0][0];
    }
}
```

寝る前の暇な時間にコードを眺めていたら、[m - 2][n - 2] のマスが常に2になることから、右端の列と底の行は全部1にしておき、それらを走査対象から外せばイテレーション内の条件分岐を無くせることに気づいた。  
ゴールのマスは基本的に正しくは 0 なのだけど、スタート地点の値 (unique path の数) の算出には基本的に影響しないので、見やすさのためにこのような実装にした。  
そうしたらコースが1マスだったとき (m == 1, n == 1) のエッジケースで正答になってくれることにも気づいたので、冒頭の if 文も削除した。  

```java
/**
 * 時間計算量: O(m * n):
 *     - O(m * n): 配列の宣言
 *     - O(m + n): 配列の底と右端の要素を 1 にする
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでのマスの位置の判定や値の計算、代入といった処理
 * 空間計算量: O(m * n): キャッシュ用の二次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] pathCountCache = new int[m][n];
        for (int i = 0; i < n; i++) {
            pathCountCache[m - 1][i] = 1;
        }
        for (int i = 0; i < m; i++) {
            pathCountCache[i][n - 1] = 1;
        }

        for (int y = m - 2; y >= 0; y--) {
            for (int x = n - 2; x >= 0; x--) {
                pathCountCache[y][x] = pathCountCache[y][x + 1] + pathCountCache[y + 1][x];
            }
        }
        return pathCountCache[0][0];
    }
}
```

一応 0 以下の値が引数に渡されたときの考慮もしておいた。

```java
/**
 * 時間計算量: O(m * n):
 *     - O(m * n): 配列の宣言
 *     - O(m + n): 配列の底と右端の要素を 1 にする
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでのマスの位置の判定や値の計算、代入といった処理
 * 空間計算量: O(m * n): キャッシュ用の二次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        if (m <= 0 || n <= 0) {
            return 0;
        }

        int[][] pathCountCache = new int[m][n];
        for (int i = 0; i < n; i++) {
            pathCountCache[m - 1][i] = 1;
        }
        for (int i = 0; i < m; i++) {
            pathCountCache[i][n - 1] = 1;
        }

        for (int y = m - 2; y >= 0; y--) {
            for (int x = n - 2; x >= 0; x--) {
                pathCountCache[y][x] = pathCountCache[y][x + 1] + pathCountCache[y + 1][x];
            }
        }
        return pathCountCache[0][0];
    }
}
```

# 63. Unique Paths II

LeetCode URL: https://leetcode.com/problems/unique-paths-ii/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

最初に思いついた愚直な再帰の実装。

```java
/**
 * かかった時間: 約12分
 * 時間計算量: O(2^(m + n)): 
 *     - 各マスで進行方向の候補が最大で2つある (進行方向が配列の boundary を超える or obstacle があるマスは候補にならない)
 *     - スタートからゴールに到達するまで m - 1 + n - 1 回の移動が必要
 * 空間計算量: O(m + n):
 *     - スタートからゴールまでの移動回数が再帰の深さになる
 *     - 移動回数は m - 1 + n - 1 回
 */
class Solution {
    int[][] obstacleGrid;
    int uniquePathCount;

    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if (obstacleGrid[0][0] == 1) {
            return 0;
        }

        this.obstacleGrid = obstacleGrid;
        this.uniquePathCount = 0;
        findPathToGoalRecursively(0, 0);
        return this.uniquePathCount;
    }

    private void findPathToGoalRecursively(int y, int x) {
        int rowCount = this.obstacleGrid.length;
        int columnCount = this.obstacleGrid[0].length;
        if (y == rowCount - 1 && x == columnCount - 1) {
            this.uniquePathCount++;
        }

        int nextColumn = x + 1;
        if (nextColumn < columnCount && this.obstacleGrid[y][nextColumn] == 0) {
            findPathToGoalRecursively(y, nextColumn);
        }
        int nextRow = y + 1;
        if (nextRow < rowCount && this.obstacleGrid[nextRow][x] == 0) {
            findPathToGoalRecursively(nextRow, x);
        }
    }
}
```

上記書いていて自然と思いつく、二次元配列のキャッシュを使った解法。

```java
/**
 * かかった時間: 約21分
 * 時間計算量: O(m * n):
 *     - O(m * n): 配列の宣言
 *     - O(m + n): 配列の一番上の行と左端列を 1 にする (途中 obstacle がある場合は中断)
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでの計算処理
 * 空間計算量: O(m * n): キャッシュ用の二次元配列
 */
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int rowCount = obstacleGrid.length, columnCount = obstacleGrid[0].length;
        int[][] uniquePathCache = new int[rowCount][columnCount];
        for (int i = 0; i < rowCount; i++) {
            if (obstacleGrid[i][0] == 1) {
                break;
            }
            uniquePathCache[i][0] = 1;
        }
        for (int i = 0; i < columnCount; i++) {
            if (obstacleGrid[0][i] == 1) {
                break;
            }
            uniquePathCache[0][i] = 1;
        }

        for (int y = 1; y < rowCount; y++) {
            for (int x = 1; x < columnCount; x++) {
                if (obstacleGrid[y][x] == 1) {
                    continue;
                }

                uniquePathCache[y][x] = uniquePathCache[y - 1][x] + uniquePathCache[y][x - 1];
            }
        }
        return uniquePathCache[rowCount - 1][columnCount - 1];
    }
}
```

さらに空間計算量を減らすため、一次元配列のキャッシュ + 左端列の最初の obstacle 出現位置を用いたキャッシュを行う。

```java
/**
 * かかった時間: 不明 (ご飯食べたりシャワー浴びながら考えていた)
 * 時間計算量: O(m * n):
 *     - O(1): 一次元配列の宣言
 *     - O(n): 配列の一番上の行を 1 にする (途中 obstacle がある場合は中断)
 *     - O(n): 左端列をスタート地点から走査し、最初に obstacle が現れる位置を特定
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでの計算処理
 * 空間計算量: O(n):
 *     - O(n): キャッシュ用の一次元配列
 *     - O(1): 左端列最初に obstacle が現れる位置
 */
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int rowCount = obstacleGrid.length;
        int columnCount = obstacleGrid[0].length;
        int[] uniquePathCache = new int[columnCount];
        for (int i = 0; i < columnCount; i++) {
            if (obstacleGrid[0][i] == 1) {
                break;
            }
            uniquePathCache[i] = 1;
        }
        int leftColumnFirstObstacleIndex = rowCount;
        for (int i = 0; i < rowCount; i++) {
            if (obstacleGrid[i][0] == 1) {
                leftColumnFirstObstacleIndex = i;
                break;
            }
        }

        for (int y = 1; y < rowCount; y++) {
            if (leftColumnFirstObstacleIndex <= y) {
                uniquePathCache[0] = 0;
            } else {
                uniquePathCache[0] = 1;
            }
            for (int x = 1; x < columnCount; x++) {
                if (obstacleGrid[y][x] == 1) {
                    uniquePathCache[x] = 0;
                    continue;
                }
                uniquePathCache[x] += uniquePathCache[x - 1];
            }
        }
        return uniquePathCache[columnCount - 1];
    }
}
```

## Step 2

### 二次元配列を用いたキャッシュ (ループをひとつにする)

他の方が書いており、こちらの方が綺麗になりそうだったので書いてみる。ついでに Obstacle を表す値 1 がマジックナンバーとなっていたので定数化。

```java
/**
 * かかった時間: 約21分
 * 時間計算量: O(m * n):
 *     - O(m * n): 配列の宣言
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでの計算処理
 * 空間計算量: O(m * n): キャッシュ用の二次元配列
 */
class Solution {
    private static final int OBSTACLE = 1;

    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int rowCount = obstacleGrid.length, columnCount = obstacleGrid[0].length;
        int[][] uniquePathCache = new int[rowCount][columnCount];
        uniquePathCache[0][0] = 1;
        for (int y = 0; y < rowCount; y++) {
            for (int x = 0; x < columnCount; x++) {
                if (obstacleGrid[y][x] == OBSTACLE) {
                    uniquePathCache[y][x] = 0;
                    continue;
                }

                if (1 <= y) {
                    uniquePathCache[y][x] += uniquePathCache[y - 1][x];
                }
                if (1 <= x) {
                    uniquePathCache[y][x] += uniquePathCache[y][x - 1];
                }
            }
        }
        return uniquePathCache[rowCount - 1][columnCount - 1];
    }
}
```

### 一次元配列を用いたキャッシュ

```java
/**
 * 時間計算量: O(m * n):
 *     - O(1): 一次元配列の宣言
 *     - O(m * n): unique path の算出を各マスで実施
 *         - O(1): 各イテレーションでの計算処理
 * 空間計算量: O(n): キャッシュ用の一次元配列
 */
class Solution {
    private static final int OBSTACLE = 1;

    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int rowCount = obstacleGrid.length, columnCount = obstacleGrid[0].length;
        int[] uniquePathCache = new int[columnCount];
        uniquePathCache[0] = 1;
        for (int y = 0; y < rowCount; y++) {
            for (int x = 0; x < columnCount; x++) {
                if (obstacleGrid[y][x] == OBSTACLE) {
                    uniquePathCache[x] = 0;
                    continue;
                }

                if (1 <= x) {
                    uniquePathCache[x] += uniquePathCache[x - 1];
                }
            }
        }
        return uniquePathCache[columnCount - 1];
    }
}
```

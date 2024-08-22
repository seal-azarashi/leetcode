# 200. Number of Islands

LeetCode URL: https://leetcode.com/problems/number-of-islands/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 22分ぐらい
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    public int numIslands(char[][] grid) {
        int numberOfIslands = 0;
        boolean[][] traversed = new boolean[grid.length][grid[0].length];
        int m = grid.length;
        int n = grid[0].length;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '0' || traversed[i][j] == true) {
                    continue;
                }

                traverseAdjacentCells(grid, traversed, m, n, i, j);
                numberOfIslands++;
            }
        }
        
        return numberOfIslands;
    }

    private void traverseAdjacentCells(char[][] grid, boolean[][] traversed, int m, int n, int i, int j) {
        boolean isOutOfBounds = i < 0 || i >= m || j < 0 || j >= n;
        if (isOutOfBounds || grid[i][j] == '0' || traversed[i][j] == true) {
            return;
        }

        traversed[i][j] = true;

        traverseAdjacentCells(grid, traversed, m, n, i + 1, j);
        traverseAdjacentCells(grid, traversed, m, n, i - 1, j);
        traverseAdjacentCells(grid, traversed, m, n, i, j - 1);
        traverseAdjacentCells(grid, traversed, m, n, i, j + 1);
    }
}
```

以下のようなことを考えて実装していました:

- 確か一度解いたことがあり、要素全てに対して走査を行う必要があったことを思い出す
- より良いアプローチが無いか考えたが、思いつかないので次のアプローチで解くことに決める:
    - 各セルを順に走査
    - 走査済リストを使い、操作中の要素が island で、かつそのリストに含まれていなければ、隣接する陸地含め全てリストに入れ、答えとなる値を increment する
    - 要素が海であればスキップ
- 要素が海で無いことを判定に入れることを忘れて数分のデバッグが必要になったが、最終的に上記のコードでパスした

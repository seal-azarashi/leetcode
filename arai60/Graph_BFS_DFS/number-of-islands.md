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
- 細かいが次のようなことも考えていた:
    - `traversed[i][j] == true` は `traversed[i][j]` とも書けるが、traversed という名前から boolean 値が格納されている値だというのは推測し辛いので、この行を読んでいる時点で traversed の型を忘れちゃってたら、若干読みやすくなるだろうなと思ってこれにした (そんなことも覚えられないぐらい疲れているなら休んだ方がいいとは思うけれど)
    - grid の領域外であることを判定するのには4つの判定が必要だが、文字数の割にあまり読む価値があると思えなかったので isOutOfBounds という変数に結果を入れてしまうことにした

# Step 2

```java
class Solution {
    public int numIslands(char[][] grid) {
        int numberOfIslands = 0;
        int m = grid.length;
        int n = grid[0].length;
        boolean[][] traversedLands = new boolean[m][n];
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '0' || traversedLands[i][j] == true) {
                    continue;
                }

                traverseAdjacentLands(grid, traversedLands, m, n, i, j);
                numberOfIslands++;
            }
        }

        return numberOfIslands;
    }

    private void traverseAdjacentLands(char[][] grid, boolean[][] traversedLands, int m, int n, int i, int j) {
        boolean isOutOfBounds = i < 0 || i >= m || j < 0 || j >= n;
        if (isOutOfBounds || grid[i][j] == '0' || traversedLands[i][j] == true) {
            return;
        }

        traversedLands[i][j] = true;

        traverseAdjacentLands(grid, traversedLands, m, n, i + 1, j);
        traverseAdjacentLands(grid, traversedLands, m, n, i - 1, j);
        traverseAdjacentLands(grid, traversedLands, m, n, i, j - 1);
        traverseAdjacentLands(grid, traversedLands, m, n, i, j + 1);
    }
}
```

- 操作対象は陸地だけであることを表すため変数、関数名を修正:
    - traversed -> traversedLands
    - traverseAdjacentCells -> traverseAdjacentLands
- traversedLands の capacity 指定に m, n を使用

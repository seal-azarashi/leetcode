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

## Step 1 の改善版

```java
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    public int numIslands(char[][] grid) {
        int islandCount = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] traversedLands = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == '0' || traversedLands[row][column] == true) {
                    continue;
                }

                traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row, column);
                islandCount++;
            }
        }

        return islandCount;
    }

    private void traverseAdjacentLands(char[][] grid, boolean[][] traversedLands, int rowCount, int columnCount, int row, int column) {
        boolean isOutOfBounds = !(row >= 0 && row < rowCount && column >= 0 && column < columnCount);
        if (isOutOfBounds || grid[row][column] == '0' || traversedLands[row][column] == true) {
            return;
        }

        traversedLands[row][column] = true;

        traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row + 1, column);
        traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row - 1, column);
        traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row, column - 1);
        traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row, column + 1);
    }
}
```

- 操作対象は陸地だけであることを表すため変数、関数名を修正:
    - traversed -> traversedLands
    - traverseAdjacentCells -> traverseAdjacentLands
- 各所で話されていた命名に関する議論を参考に、m, n, i, j を rowCount, columnCount, row, column に修正
- oda さんがどこかで指摘されてたように isOutOfBounds の定義を修正
- 引数 i, j を row, column に修正 ([参考](https://github.com/nittoco/leetcode/pull/23#discussion_r1667241892))
- traversedLands の capacity 指定に m, n を使用

## 再帰関数の代わりにスタックを使う

~~(Leetcode の constraints ではそうならないとは書いてあるものの) 大きな grid が渡されたらスタックオーバーフローになる可能性があるため、スタックを用いたより安全な実装にしてみる。~~ 👈 誤りです ([参考](https://github.com/seal-azarashi/leetcode/pull/17#discussion_r1733611615))

```java
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private record Cell(int row, int column) {};

    public int numIslands(char[][] grid) {
        int islandCount = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] traversedLands = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == '0' || traversedLands[row][column] == true) {
                    continue;
                }

                traverseAdjacentLands(grid, traversedLands, rowCount, columnCount, row, column);
                islandCount++;
            }
        }

        return islandCount;
    }

    private void traverseAdjacentLands(char[][] grid, boolean[][] traversedLands, int rowCount, int columnCount, int row, int column) {
        Deque<Cell> traversingCells = new ArrayDeque<>();
        traversingCells.add(new Cell(row, column));
        while (!traversingCells.isEmpty()) {
            Cell cell = traversingCells.removeFirst();
            boolean isOutOfBounds = !(cell.row >= 0 && cell.row < rowCount && cell.column >= 0 && cell.column < columnCount);
            if (isOutOfBounds || grid[cell.row][cell.column] == '0' || traversedLands[cell.row][cell.column] == true) {
                continue;
            }

            traversedLands[cell.row][cell.column] = true;

            traversingCells.add(new Cell(cell.row + 1, cell.column));
            traversingCells.add(new Cell(cell.row - 1, cell.column));
            traversingCells.add(new Cell(cell.row, cell.column + 1));
            traversingCells.add(new Cell(cell.row, cell.column - 1));
        }
    }
}
```

- "This class is likely to be faster than Stack when used as a stack" とある ArrayDeque を Stack クラスの代わりに使ってみた
    - nodchip さんも言及されてた: https://github.com/goto-untrapped/Arai60/pull/38#discussion_r1683649014
        - リングバッファ (Circular Buffer とも言うのね) だったのか
- ArrayDeque に initial capacity を設定すると、ケースによっては劇的に遅くなった (処理時間が7倍ほどになった) ことが興味深かった
    - CPU キャッシュに乗らなくなるためだろうか？
    - しかしメモリ消費量は1~2割ほど削減される
- 再帰関数を使った場合と比べおよそ2倍の処理時間がかかっている
    - スタック的な役割のオブジェクトの利用よりも、スタックフレームの利用の方がよっぽど最適化されているということなのだろうか？
    - 他に致命的な要因が無いかと record の代わりに整数型配列を使ったりもしたが、それほど大きな影響はなかった

## 走査時に grid の値を書き換える解法

[nodchip さんのコメント](https://github.com/thonda28/leetcode/pull/15#discussion_r1685703323)を見て、なるほど面白いなと思って書いてみました。実行速度が最も速い。  
Leetcode の submission 時に計測される Runtime と Memory も良くなっていました。しかしコメントにも違和感として指摘されていますが、入力値を書き換えてしまう副作用を許容できるのかについては、ちゃんと確認を取りながら実装した方がいいかなと思います。

```java
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    public int numIslands(char[][] grid) {
        int islandCount = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == '0') {
                    continue;
                }

                traverseAdjacentLands(grid, rowCount, columnCount, row, column);
                islandCount++;
            }
        }

        return islandCount;
    }

    private void traverseAdjacentLands(char[][] grid, int rowCount, int columnCount, int row, int column) {
        boolean isOutOfBounds = !(row >= 0 && row < rowCount && column >= 0 && column < columnCount);
        if (isOutOfBounds || grid[row][column] == '0') {
            return;
        }

        grid[row][column] = '0';

        traverseAdjacentLands(grid, rowCount, columnCount, row + 1, column);
        traverseAdjacentLands(grid, rowCount, columnCount, row - 1, column);
        traverseAdjacentLands(grid, rowCount, columnCount, row, column - 1);
        traverseAdjacentLands(grid, rowCount, columnCount, row, column + 1);
    }
}
```

## Union-find を用いた解法

他の方々が Union-find を使っていたので私もやってみました。

```java
// 時間計算量 (union 操作は定数時間とみなす): O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private class UnionFindIslands {
        private int[] parent;
        private int[] rank;
        private int count;

        private UnionFindIslands(char[][] grid) {
            int rowCount = grid.length;
            int columnCount = grid[0].length;
            parent = new int[rowCount * columnCount];
            rank = new int[rowCount * columnCount];
            count = 0;
            for (int row = 0; row < rowCount; row++) {
                for (int column = 0; column < columnCount; column++) {
                    if (grid[row][column] == '0') {
                        continue;
                    }

                    int cell = row * columnCount + column;
                    parent[cell] = cell;
                    count++;
                }
            }
        }

        private void union(int x, int y) {
            int xRoot = find(x);
            int yRoot = find(y);

            if (xRoot == yRoot) {
                return;
            }

            if (rank[xRoot] < rank[yRoot]) {
                parent[xRoot] = yRoot;
            } else if (rank[xRoot] > rank[yRoot]) {
                parent[yRoot] = xRoot;
            } else {
                parent[yRoot] = xRoot;
                rank[xRoot]++;
            }
            count--;
        }

        private int find(int i) {
            if (parent[i] == i) {
                return parent[i];
            }
            return find(parent[i]);
        }

        private int getCount() {
            return count;
        }
    }

    public int numIslands(char[][] grid) {
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        UnionFindIslands unionFindIslands = new UnionFindIslands(grid);

        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == '0') {
                    continue;
                }

                grid[row][column] = '0';

                int coordinate = row * columnCount + column;
                if (row + 1 < rowCount && grid[row + 1][column] == '1') {
                    unionFindIslands.union(coordinate, (row + 1) * columnCount + column);
                }
                if (column + 1 < columnCount && grid[row][column + 1] == '1') {
                    unionFindIslands.union(coordinate, coordinate + 1);
                }
            }
        }

        return unionFindIslands.getCount();
    }
}
```

- parent, rank は二次元配列にも出来るが、連続したメモリ領域が使用されるため処理の効率化が見込める一次元配列を採用した

# Step 3

次の理由で再帰と走査済 boolean 配列を用いた実装を選びました:

- 処理が速い
- ~~Leetcode の constraints 上スタックオーバーフローが起きる可能性は極めて低い~~ 👈 誤りです ([参考](https://github.com/seal-azarashi/leetcode/pull/17#discussion_r1733611615))
- 副作用を生まない

```java
// 解いた時間: 8分ぐらい
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    public int numIslands(char[][] grid) {
        int islandCount = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] traversedLands = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == '0' || traversedLands[row][column] == true) {
                    continue;
                }

                traverseLands(grid, rowCount, columnCount, traversedLands, row, column);
                islandCount++;
            }
        }

        return islandCount;
    }

    private void traverseLands(char[][] grid, int rowCount, int columnCount, boolean[][] traversedLands, int row, int column) {
        boolean isOutOfBounds = !(row >= 0 && row < rowCount && column >= 0 && column < columnCount);
        if (isOutOfBounds || grid[row][column] == '0' || traversedLands[row][column] == true) {
            return;
        }

        traversedLands[row][column] = true;

        traverseLands(grid, rowCount, columnCount, traversedLands, row + 1, column);
        traverseLands(grid, rowCount, columnCount, traversedLands, row - 1, column);
        traverseLands(grid, rowCount, columnCount, traversedLands, row, column + 1);
        traverseLands(grid, rowCount, columnCount, traversedLands, row, column - 1);
    }
}
```

# Step 4

## 再帰関数の代わりにスタックを使う

```java
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private char WATER = '0';
    private record Cell(int row, int column) {};

    public int numIslands(char[][] grid) {
        int islandCount = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] isVisited = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == WATER || isVisited[row][column] == true) {
                    continue;
                }

                traverseAdjacentLands(grid, isVisited, rowCount, columnCount, row, column);
                islandCount++;
            }
        }

        return islandCount;
    }

    private void traverseAdjacentLands(char[][] grid, boolean[][] isVisited, int rowCount, int columnCount, int row, int column) {
        Deque<Cell> traversingCells = new ArrayDeque<>();
        traversingCells.addFirst(new Cell(row, column));
        while (!traversingCells.isEmpty()) {
            Cell cell = traversingCells.removeFirst();
            boolean isInsideGrid = 0 <= cell.row && cell.row < rowCount && 0 <= cell.column && cell.column < columnCount;
            if (!isInsideGrid || grid[cell.row][cell.column] == WATER || isVisited[cell.row][cell.column] == true) {
                continue;
            }

            isVisited[cell.row][cell.column] = true;

            traversingCells.addFirst(new Cell(cell.row + 1, cell.column));
            traversingCells.addFirst(new Cell(cell.row - 1, cell.column));
            traversingCells.addFirst(new Cell(cell.row, cell.column + 1));
            traversingCells.addFirst(new Cell(cell.row, cell.column - 1));
        }
    }
}
```

- [nodchip さんのレビュー](https://github.com/seal-azarashi/leetcode/pull/17#discussion_r1732478049)で再帰関数を用いた実装だとスタックオーバーフローになる可能性があることに気づいたため、スタックを用いた解法を第一の選択肢にした
- 同じく [nodchip さんのレビュー](https://github.com/seal-azarashi/leetcode/pull/17#discussion_r1732474947)を参考に traverseAdjacentLands() 内のにある条件判定をリファクタリングした
    - 肯定的な意味合いの boolean 値にし、判定時に否定演算子を付けて用いるようにした
    - 数直線上に一直線上に並べるような並びにした

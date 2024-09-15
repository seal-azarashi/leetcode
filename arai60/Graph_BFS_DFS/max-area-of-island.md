# 695. Max Area of Island

LeetCode URL: https://leetcode.com/problems/max-area-of-island/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 20分ぐらい
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private int WATER = 0;
    private record Cell(int row, int column) {};

    public int maxAreaOfIsland(int[][] grid) {
        int maxAreaOfIsland = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] isVisited = new boolean[rowCount][columnCount];
        int[] areaOfIsland = new int[1];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == WATER || isVisited[row][column]) {
                    continue;
                }

                areaOfIsland[0] = 0;
                traverseIslands(grid, areaOfIsland, isVisited, row, column);
                if (areaOfIsland[0] > maxAreaOfIsland) {
                    maxAreaOfIsland = areaOfIsland[0];
                }
            }
        }

        return maxAreaOfIsland;
    }

    private void traverseIslands(int[][] grid, int[] areaOfIsland, boolean[][] isVisited, int row, int column) {
        Deque<Cell> cells = new ArrayDeque<>();
        cells.push(new Cell(row, column));
        while (!cells.isEmpty()) {
            Cell cell = cells.pop();
            boolean isInsideGrid = 0 <= cell.row && cell.row < grid.length && 0 <= cell.column && cell.column < grid[0].length;
            if (!isInsideGrid || grid[cell.row][cell.column] == WATER || isVisited[cell.row][cell.column]) {
                continue;
            }

            isVisited[cell.row][cell.column] = true;
            areaOfIsland[0]++;

            cells.push(new Cell(cell.row + 1, cell.column));
            cells.push(new Cell(cell.row - 1, cell.column));
            cells.push(new Cell(cell.row, cell.column + 1));
            cells.push(new Cell(cell.row, cell.column - 1));
        }
    }
}
```

以下のようなことを考えて実装していました:

- 一つ前に解いた問題とほぼ同じアプローチで解けると考え、実際その通りになった: https://leetcode.com/problems/number-of-islands/description/
- 一般的にオブジェクトの生成は思い処理とされるが、 JVM が賢く動いてくれるからか、少なくとも Leetcode の実行処理結果を見る限りは気にするほどの差が生まれなかったため、より可読性を向上できると思い、 cells や Cell record については都度生成するような実装にした
    - むしろ cells については maxAreaOfIsland() で生成して、直前に clear メソッドを実行してから traverseIslands() に渡すようにすると、処理時間がより多くかかっていて興味深かった
    - clear メソッドの実行には O(n) かかるので、これが new を実行するよりも、結果として多くの時間を要したのだろうか？
- 他にも再帰関数を使う、 isVisited を用いる代わりに grid の値を書き換える、 Union-find を用いるといったアプローチがあるが、今回のようにスタックを使うような処理が次の理由でベストだと考えた:
    - 再帰関数だとスタックオーバーフローになり得る
        - 今回は grid のサイズは最大でも 50*50 だと問題文の constraints に明記されており、だいたい 10k ぐらいまでの再帰なら行えると考えられる Java では今回は問題にならないと考えられるが、あくまで一般論として
    - grid の値を書き換えるのはとりわけ空間計算量を減らせるという観点で優れているが、引数に受け取った値の中身を書き換える副作用を発生させることが許容されるのは稀だと考えており、第一の選択肢にはならないと考える
    - Union-find の実装をするのはそれなりにコストがかかり、かつこの問題に対するアプローチとしては他のものよりも素直さに欠ける印象がある

# Step 2

## Step 1 の解法の修正版

```java
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private static final int WATER = 0;
    private record Cell(int row, int column) {};

    public int maxAreaOfIsland(int[][] grid) {
        int maxAreaOfIsland = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] isVisited = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == WATER || isVisited[row][column]) {
                    continue;
                }

                int areaOfIsland = traverseIsland(grid, isVisited, row, column);
                if (areaOfIsland > maxAreaOfIsland) {
                    maxAreaOfIsland = areaOfIsland;
                }
            }
        }

        return maxAreaOfIsland;
    }

    private int traverseIsland(int[][] grid, boolean[][] isVisited, int row, int column) {
        int areaOfIsland = 0;
        Deque<Cell> cells = new ArrayDeque<>();
        cells.push(new Cell(row, column));
        while (!cells.isEmpty()) {
            Cell cell = cells.pop();
            boolean isInsideGrid = 0 <= cell.row && cell.row < grid.length && 0 <= cell.column && cell.column < grid[0].length;
            if (!isInsideGrid || grid[cell.row][cell.column] == WATER || isVisited[cell.row][cell.column]) {
                continue;
            }

            isVisited[cell.row][cell.column] = true;
            areaOfIsland++;

            cells.push(new Cell(cell.row + 1, cell.column));
            cells.push(new Cell(cell.row - 1, cell.column));
            cells.push(new Cell(cell.row, cell.column + 1));
            cells.push(new Cell(cell.row, cell.column - 1));
        }

        return areaOfIsland;
    }
}
```

- WATER は定数として扱うので static, final 修飾子を付与: https://github.com/goto-untrapped/Arai60/pull/31#discussion_r1650111955
- 関数 traverseIslands() について、これは一度に最大でひとつの島しか走査しないので、単数形の traverseIsland() に名前を修正
- traverseIsland() は Step 1 のように引数に受け取った areaOfIsland を書き換えるのではなく、より素直に思える、関数内で計算した島の area の大きさを返すような実装に修正

## その他

再帰関数を使う、 isVisited を用いる代わりに grid の値を書き換える、 Union-find を用いるといったアプローチが思いつきましたが、類題で実施した内容とほぼ同じようなアウトプットになることが予想されたので、今回は実装を見送り、その分別の時間を解くのに問題を使おうと考えました。  
類題で実施した内容はこちらの Step 2 にまとまっています: https://github.com/seal-azarashi/leetcode/pull/17/files?short_path=5a321eb#diff-5a321eb303c17e1955dbf9cf8de8318afdd9ff3496f0d55b36327caa0fb1682e

いくつか気になったレビューについての感想を書き残しておきます:

- 講師陣の皆さんがどのぐらい時間を書けて解かれていたのかを知れて勉強になった: https://github.com/TORUS0818/leetcode/pull/20/files#r1657723196
- Java は1万回ぐらいなら再帰できる、それはなぜか？と聞かれて自分なら答えられるだろうか?: https://github.com/goto-untrapped/Arai60/pull/31#discussion_r1650006241

# Step 3

```java
// 解いた時間: 13分ぐらい
// 時間計算量: O(m * n)
// 空間計算量: O(m * n)
class Solution {
    private static final int WATER = 0;
    private record Cell(int row, int column) {};

    public int maxAreaOfIsland(int[][] grid) {
        int maxAreaOfIsland = 0;
        int rowCount = grid.length;
        int columnCount = grid[0].length;
        boolean[][] isVisited = new boolean[rowCount][columnCount];
        for (int row = 0; row < rowCount; row++) {
            for (int column = 0; column < columnCount; column++) {
                if (grid[row][column] == WATER || isVisited[row][column]) {
                    continue;
                }

                int areaOfIsland = calculateAreaOfIsland(grid, isVisited, row, column);
                if (areaOfIsland > maxAreaOfIsland) {
                    maxAreaOfIsland = areaOfIsland;
                }
            }
        }

        return maxAreaOfIsland;
    }

    private int calculateAreaOfIsland(int[][] grid, boolean[][] isVisited, int row, int column) {
        int areaOfIsland = 0;
        Deque<Cell> cells = new ArrayDeque<>();
        cells.push(new Cell(row, column));
        while (!cells.isEmpty()) {
            Cell cell = cells.pop();
            boolean isInsideGrid = 0 <= cell.row && cell.row < grid.length && 0 <= cell.column && cell.column < grid[0].length;
            if (!isInsideGrid || grid[cell.row][cell.column] == WATER || isVisited[cell.row][cell.column]) {
                continue;
            }

            isVisited[cell.row][cell.column] = true;
            areaOfIsland++;

            cells.push(new Cell(cell.row + 1, cell.column));
            cells.push(new Cell(cell.row - 1, cell.column));
            cells.push(new Cell(cell.row, cell.column + 1));
            cells.push(new Cell(cell.row, cell.column - 1));
        }

        return areaOfIsland;
    }
}
```

- 無意識で関数 traverseIsland() を calculateAreaOfIsland() と名付けていた
    - こちらのほうが役割をより明確に表していて良いと思った

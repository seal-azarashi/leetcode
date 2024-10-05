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
(💭 後で気付きましたが、スタートからの走査でも unique path のキャッシュは可能ですね。 Step 2 でそちらの実装もしてみます。)  
(💭 エッジケースの考慮に苦労しました。 Leetcode のテストケースが用意されていないとここまで作り込めなかった気がしますね... 採点ツールを使わない、ホワイトボードに書くような形式の面接だと不合格となってしまわないか不安がよぎりました)  

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

(💭 夜中にダラダラやってたのでかかった時間は15分を全然超えてますが、意外と出来るものですね。ちゃんと考えを整理しながら解法を段階的にアップデートしていって理解が深まったので、今後も長い間覚えていられそうです。)  

## Step 2

### スタート地点から走査する

Step 1 の解法の走査を、スタート地点から行うように書き直してみる。

```java
class Solution {
    public int uniquePaths(int m, int n) {
        if (m <= 0 || n <= 0) {
            return 0;
        }

        int[][] pathCountCache = new int[m][n];
        for (int i = 0; i < n; i++) {
            pathCountCache[0][i] = 1;
        }
        for (int i = 0; i < m; i++) {
            pathCountCache[i][0] = 1;
        }

        for (int y = 1; y < m; y++) {
            for (int x = 1; x < n; x++) {
                pathCountCache[y][x] = pathCountCache[y - 1][x] + pathCountCache[y][x - 1];
            }
        }
        return pathCountCache[m - 1][n - 1];
    }
}
```

### 1次元配列にキャッシュする

解答を見てみると、一次元の配列をキャッシュに使うものがあったので、これを踏まえて step 1 のコードを改良する。  
step 1 では次のような2次元配列を使っていたが、まず右端の列については全て 1 であることがキャッシュに残っていれば良いので、1列分の情報量は必要ない。

```
        ┌ この要素たちはひとつで充分
0, 0, 1 ┤
3, 2, 1 ┤
1, 1, 1 ┘
```

他の列の情報も全行分保持する必要はなく、イテレーションごとに最新の (一番下の行の) 値さえあればいい。

```
0, 0, 1
3, 2, 1
1, 1, 1 <- 次に計算する行 (一番上の行) の計算にはもうこの行の情報は必要ない
```

上記の例だと、 [3, 2, 1] の一次元配列があれば、この右端以外 (この場合 0-1) の要素をアップデートしていくことで step 1 と同等の処理が実現できる。  
これを踏まえて以下のように書き直した。

```java
/**
 * 時間計算量: O(m * n):
 *     - O(1): 配列の宣言 (一次元配列のためのメモリ確保処理が最適化されている想定)
 *     - O(n): 配列の要素を全て 1 にする
 *     - O(m * n): unique path の算出を実施
 *         - O(1): 各イテレーションでの処理
 * 空間計算量: O(n): キャッシュ用の一次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        if (m <= 0 || n <= 0) {
            return 0;
        }

        int[] uniquePathCache = new int[n];
        Arrays.fill(uniquePathCache, 1);
        for (int y = m - 2; y >= 0; y--)  {
            for (int x = n - 2; x >= 0; x--) {
                uniquePathCache[x] += uniquePathCache[x + 1];
            }
        }
        return uniquePathCache[0];
    }
}
```

m, n の小さい方を配列のサイズに指定すれば、配列の初期化処理と空間計算量が O(n) から O(min(m, n)) になる。 

```java
/**
 * 時間計算量: O(m * n):
 *     - O(1): 配列の宣言 (一次元配列のためのメモリ確保処理が最適化されている想定)
 *     - O(min(m, n)): 配列の要素を全て 1 にする
 *     - O(m * n): unique path の算出を実施
 *         - O(1): 各イテレーションでの処理
 * 空間計算量: O(min(m, n)): キャッシュ用の一次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        if (m <= 0 || n <= 0) {
            return 0;
        }

        // Make Space Complexity O(min(m, n)) from O(n)
        if (m < n) {
            return uniquePaths(n, m);
        }

        int[] uniquePathCache = new int[n];
        Arrays.fill(uniquePathCache, 1);
        for (int y = m - 2; y >= 0; y--)  {
            for (int x = n - 2; x >= 0; x--) {
                uniquePathCache[x] += uniquePathCache[x + 1];
            }
        }
        return uniquePathCache[0];
    }
}
```

### 数学的の組み合わせの問題として計算

[組み合わせの公式](https://manabitimes.jp/math/1352#4)を使って解く。 Constraints の範囲内でも最大で 198! を扱うことになり、その結果は int の最大値 2^31 - 1 を簡単に超えてしまう (int が扱えるのは 12! まで)。そのため java.math パッケージの BigInteger を使う。  
(💭 [Math クラス](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Math.html) や [math パッケージ](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/package-summary.html) を眺めてみましたが、意外と factorial や combination 算出用の関数はないんですね)  
(💭 util や lang 以外のパッケージを使うのは久々です。これら以外は LeetCode では明示的にインポートする必要があるので import 文も久々に書きました。)  

```java
import java.math.BigInteger;

class Solution {
    public int uniquePaths(int m, int n) {
        return combination(m + n - 2, n - 1).intValue();
    }

    private BigInteger combination(int m, int n) {
        return factorial(m).divide(factorial(n).multiply(factorial(m - n)));
    }

    private BigInteger factorial(int m) {
        BigInteger result = BigInteger.ONE;
        for (int i = 2; i <= m; i++) {
            result = result.multiply(BigInteger.valueOf(i));
        }
        return result;
    }
}
```

## Step 3

一次元配列にキャッシュする案を採用。加えてスタート地点から操作する方が直感的に感じたのでそのようにした。  
尚、 m, n からより小さい値を選んで計算量を最適化するのは、次の考えから、与えられた問題に対してはやり過ぎだと判断している:  

- ユースケースは、二次元のグリッドでなにかをするボードゲームか何かで、移動パターンの数を算出する処理か何かと考えられる
- この最適化の恩恵が受けられるほどの大きさのグリッド内での移動をする場合、そのパターン数を算出したくなることは無いのではないかと考えた (💭 全然ゲーム詳しくないので、間違っていたらご指摘ください)

なので上記については、面接官との会話を通して、必要だと判断したら実装に加えたい。

```java
/**
 * 解いた時間: 約3分
 * 時間計算量: O(m * n):
 *     - O(1): 配列の宣言 (一次元配列のためのメモリ確保処理が最適化されている想定)
 *     - O(n): 配列の要素を全て 1 にする
 *     - O(m * n): unique path の算出を実施
 *         - O(1): 各イテレーションでの処理
 * 空間計算量: O(n): キャッシュ用の一次元配列
 */
class Solution {
    public int uniquePaths(int m, int n) {
        if (m <= 0 || n <= 0) {
            return 0;
        }

        int[] uniquePathCache = new int[n];
        Arrays.fill(uniquePathCache, 1);
        for (int y = 1; y < m; y++) {
            for (int x = 1; x < n; x++) {
                uniquePathCache[x] += uniquePathCache[x - 1];
            }
        }
        return uniquePathCache[n - 1];
    }
}
```

oda さんが言うところの[知ってたとみなしての採点対象外](https://discord.com/channels/1084280443945353267/1206101582861697046/1207405733667410051)とならないよう、面接では、しっかりとどうやってたどり着いたのかは説明しながら書いていきたいと思った。  

> いや、私、何を言っているかというと、これ、Kadane's algorithm といって、名前がついている上に常識に入っていない程度のものなので、いきなりこれを書いたら、Kadane 程度の天才か、知っていたんだろうなって思って、この問題は採点外にして、次の問題を出します。  
> from: https://discord.com/channels/1084280443945353267/1206101582861697046/1207405733667410051

(💭 この問題ではそんなに難しいことしてないのでそうはならないかもしれませんが、今後の心構えとして。)  

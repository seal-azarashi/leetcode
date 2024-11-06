# 322. Coin Change

LeetCode URL: https://leetcode.com/problems/coin-change/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

まずは愚直にあらゆる組み合わせ (coin の合算値が amount を超えるものは除く) を出す方法を考えつくが、計算量の面から好ましくないので別のアプローチが無いか探したい気持ちになる。  
すぐに次のアプローチが思いついたので、以下の通り実装した: 各インデックスの数字と同じ数値の額を何枚のコインで作れるかを保持する配列を作り、各要素を更新する。1 から amount までの各整数値ごとに coins の要素数だけ計算が必要なので計算量は O(n * m) で悪くないはず。  

```java
/**
 * 時間計算量: O(n * m): 
 *     - amount + 1 回処理を実行: O(n)
 *         - coins の要素数だけ処理を実行: O(m)
 *             - 条件判定といった定数計算量の操作: O(1)
 * 空間計算量: O(n):
 *     - キャッシュ用配列: O(n)
 *     - その他定数計算量の変数等: O(1)
 * 
 * ※ n = amount, m = coins.length
 */
class Solution {
    private final int CANNOT_BE_MADE_UP = -1;

    public int coinChange(int[] coins, int amount) {
        /**
         * 各インデックスの数字と同じ数値の額を何枚のコインで作れるかを保持する。
         * 計算の都合上、額が0の際は必要な枚数も0であるという情報が欲しいので、そのために要素数を + 1 している。
         */
        int[] minCoinCountToMakeUp = new int[amount + 1];

        for (int i = 1; i < minCoinCountToMakeUp.length; i++) {
            int minCount = Integer.MAX_VALUE;
            boolean isAnyCountFound = false;
            for (int coin : coins) {
                int remainder = i - coin;
                if (remainder < 0 || minCoinCountToMakeUp[remainder] == CANNOT_BE_MADE_UP) {
                    continue;
                }

                minCount = Math.min(minCount, minCoinCountToMakeUp[remainder]);
                isAnyCountFound = true;
            }
            if (!isAnyCountFound) {
                minCoinCountToMakeUp[i] = CANNOT_BE_MADE_UP;
                continue;
            }

            minCoinCountToMakeUp[i] = minCount + 1;
        }
        
        return minCoinCountToMakeUp[amount];
    }
}
```

## Step 2

### 1 から amount まで、それぞれ何枚のコインで作れるかをキャッシュする DP (Step 1 と同じ)

- ちょっとパズル感がある (下の方まで処理を読まないと各要素が何を表すのか想像しづらい) 印象があったので、キャッシュ用配列の宣言時にいくつか処理を寄せ、コードコメントを追加した。コードコメントの一部は javadoc に書いてもいいかもしれない。
- 変数名もいくつかより理解しやすそうなものに修正している。
- Leetcode 上でなく実務での利用を想定して null チェックも入れた。
- minCoinCount の値は `amount + 1` でも良いというコメントがあったが、その数値が何なのかを読み手が理解しなければならないことに抵抗があったので採用を見送った。既存の Integer.MAX_VALUE も全く理解が必要ないわけでは無いが、より素直に思える。

```java
/**
 * 時間計算量: O(n * m): 
 *     - キャッシュ用配列の作成: O(n)
 *     - amount + 1 回処理を実行: O(n)
 *         - coins の要素数だけ処理を実行: O(m)
 *             - 条件判定といった定数計算量の操作: O(1)
 * 空間計算量: O(n):
 *     - キャッシュ用配列: O(n)
 *     - その他定数計算量の変数等: O(1)
 * 
 * ※ n = amount, m = coins.length
 */
class Solution {
    private final int NO_COMBINATION = -1;

    public int coinChange(int[] coins, int amount) {
        Objects.requireNonNull(coins, "Argument coins must not be null");

        /**
         * 各インデックスの数字と同じ数値の額を何枚のコインで作れるかを保持する。
         * 計算の都合上、額が0の際は必要な枚数も0であるという情報が欲しいので、そのために要素数を + 1 している。
         * インデックス 0 の初期値は 0 で、他は計算の都合上、その額が作れないことを示す値を入れておく。作れることが判明次第更新される。
         */
        int[] makeUpAmountToCoinCount = new int[amount + 1];
        Arrays.fill(makeUpAmountToCoinCount, NO_COMBINATION);
        makeUpAmountToCoinCount[0] = 0;

        for (int currentAmount = 1; currentAmount < makeUpAmountToCoinCount.length; currentAmount++) {
            int minCoinCount = Integer.MAX_VALUE;
            boolean isAnyCombinationFound = false;
            for (int coin : coins) {
                int remainder = currentAmount - coin;
                if (remainder < 0 || makeUpAmountToCoinCount[remainder] == NO_COMBINATION) {
                    continue;
                }

                int coinCountToMakeUpWithTheCoin = makeUpAmountToCoinCount[remainder] + 1;
                minCoinCount = Math.min(minCoinCount, coinCountToMakeUpWithTheCoin);
                isAnyCombinationFound = true;
            }
            if (isAnyCombinationFound) {
                makeUpAmountToCoinCount[currentAmount] = minCoinCount;
            }
        }
        return makeUpAmountToCoinCount[amount];
    }
}
```

### BFS

他の方の解法にあったので書いてみる。TLE。  
DP の解法より遅い理由は、record オブジェクトの作成やスタックの操作にメモリのランダムアクセスが行われるからだろうか。まだ改善は出来そうだが一旦ここまで。

```java
/**
* 時間計算量: O(n * m): 
*     - 入力コインの前処理: O(m)
*     - キャッシュ用配列の作成: O(n)
*     - キューを使用した探索: O(n * m)
*         - 最悪の場合、各 amount に対して全てのコインを試す
* 空間計算量: O(n * m):
*     - キャッシュ用配列: O(n)
*     - キュー: 最悪の場合 O(n * m)
*     - その他定数計算量の変数等: O(1)
* 
* ※ n = amount, m = coins.length
*/
class Solution {
    private final int NO_COMBINATION = -1;

    private record CoinState(int numCoins, int currentAmount) {}

    public int coinChange(int[] coins, int amount) {
        Objects.requireNonNull(coins, "Argument coins must not be null");
        
        int[] filteredCoins = Arrays.stream(coins)
                .filter(coin -> coin <= amount)
                .toArray();

        int[] makeUpAmountToCoinCount = new int[amount + 1];
        Arrays.fill(makeUpAmountToCoinCount, Integer.MAX_VALUE);
        
        Queue<CoinState> coinStateQueue = new ArrayDeque<>();
        coinStateQueue.offer(new CoinState(0, 0));
        
        while (!coinStateQueue.isEmpty()) {
            CoinState coinState = coinStateQueue.poll();
            makeUpAmountToCoinCount[coinState.currentAmount()] = Math.min(
                makeUpAmountToCoinCount[coinState.currentAmount()],
                coinState.numCoins()
            );
            
            for (int coin : filteredCoins) {
                int nextAmount = coinState.currentAmount() + coin;
                if (nextAmount > amount) {
                    continue;
                }
                if (makeUpAmountToCoinCount[nextAmount] <= coinState.numCoins() + 1) {
                    continue;
                }
                coinStateQueue.offer(new CoinState(coinState.numCoins() + 1, nextAmount));
            }
        }
        
        return makeUpAmountToCoinCount[amount] == Integer.MAX_VALUE 
            ? NO_COMBINATION 
            : makeUpAmountToCoinCount[amount];
    }
}
```

### 非再帰の DFS

他の方の解法にあったので書いてみる。実行時間が上の解法より10倍以上長いが TLE にはならなかった。BFS の方が効率は良さそうに思えたが、テストケースとの相性がたまたまよかったのだろうか。

```java
/**
 * 時間計算量: O(n * m): 
 *     - 入力コインの前処理: O(m)
 *     - キャッシュ用配列の作成: O(n)
 *     - スタックを使用した探索: O(n * m)
 *         - 最悪の場合、各 amount に対して全てのコインを試す
 * 空間計算量: O(n * m):
 *     - キャッシュ用配列: O(n)
 *     - スタック: 最悪の場合 O(n * m)
 *     - その他定数計算量の変数等: O(1)
 * 
 * ※ n = amount, m = coins.length
 */
class Solution {
    private final int NO_COMBINATION = -1;

    private record CoinState(int numCoins, int currentAmount) {}

    public int coinChange(int[] coins, int amount) {
        Objects.requireNonNull(coins, "Argument coins must not be null");

        int[] filteredCoins = Arrays.stream(coins)
                .filter(coin -> coin <= amount)
                .toArray();

        int[] makeUpAmountToCoinCount = new int[amount + 1];
        Arrays.fill(makeUpAmountToCoinCount, Integer.MAX_VALUE);
        
        Stack<CoinState> coinStateStack = new Stack<>();
        coinStateStack.push(new CoinState(0, 0));
        
        while (!coinStateStack.isEmpty()) {
            CoinState coinState = coinStateStack.pop();
            makeUpAmountToCoinCount[coinState.currentAmount()] = Math.min(
                makeUpAmountToCoinCount[coinState.currentAmount()],
                coinState.numCoins()
            );
            for (int coin : filteredCoins) {
                int nextAmount = coinState.currentAmount() + coin;
                if (nextAmount > amount) {
                    continue;
                }
                if (makeUpAmountToCoinCount[nextAmount] <= coinState.numCoins() + 1) {
                    continue;
                }
                coinStateStack.push(new CoinState(coinState.numCoins() + 1, nextAmount));
            }
        }
        
        return makeUpAmountToCoinCount[amount] == Integer.MAX_VALUE 
            ? NO_COMBINATION 
            : makeUpAmountToCoinCount[amount];
    }
}
```

### 再帰の DFS

スタックオーバーフローのリスクがあるので優先して選ぶことはないですが、再帰による実装も書いてみます。

```java
/**
 * 時間計算量: O(n * m): 
 *     - 入力コインの前処理: O(m)
 *     - キャッシュ用配列の作成: O(n)
 *     - 再帰的な探索: O(n * m)
 *         - 最悪の場合、各 amount に対して全てのコインを試す
 * 空間計算量: O(n):
 *     - キャッシュ用配列: O(n)
 *     - 再帰スタック: 最悪の場合 O(n)
 *     - その他定数計算量の変数等: O(1)
 * 
 * ※ n = amount, m  = coins.length
 */
class Solution {
    private final int NO_COMBINATION = -1;

    public int coinChange(int[] coins, int amount) {
        Objects.requireNonNull(coins, "Argument coins must not be null");
        
        int[] filteredCoins = Arrays.stream(coins)
                .filter(coin -> coin <= amount)
                .toArray();

        int[] makeUpAmountToCoinCount = new int[amount + 1];
        Arrays.fill(makeUpAmountToCoinCount, Integer.MAX_VALUE);
        makeUpAmountToCoinCount[0] = 0;
        
        updateMinCoinCountsRecursively(filteredCoins, amount, 0, 0, makeUpAmountToCoinCount);
        
        return makeUpAmountToCoinCount[amount] == Integer.MAX_VALUE 
            ? NO_COMBINATION 
            : makeUpAmountToCoinCount[amount];
    }
    
    private void updateMinCoinCountsRecursively(
            int[] coins,
            int targetAmount,
            int currentAmount,
            int numCoins,
            int[] makeUpAmountToCoinCount
    ) {
        makeUpAmountToCoinCount[currentAmount] = Math.min(makeUpAmountToCoinCount[currentAmount], numCoins);
            
        for (int coin : coins) {
            int nextAmount = currentAmount + coin;
            if (nextAmount > targetAmount) {
                continue;
            }
            if (makeUpAmountToCoinCount[nextAmount] <= numCoins + 1) {
                continue;
            }
            updateMinCoinCountsRecursively(coins, targetAmount, nextAmount, numCoins + 1, makeUpAmountToCoinCount);
        }
    }
}
```

## Step 3

実装のシンプルさと処理効率から Step 2 に書いた DP の解法を選びました。このステップでは5分以内に書くことを目指すのでコメントや引数の null チェックは省略していますが、実際の面接ではそういったものが必要であることをしっかり補足したいところです。

```java
/**
 * 解いた時間: 約5分
 * 時間計算量: O(n * m): 
 *     - 入力コインの前処理: O(m)
 *     - キャッシュ用配列の作成: O(n)
 *     - キューを使用した探索: O(n * m)
 *         - 最悪の場合、各 amount に対して全てのコインを試す
 * 空間計算量: O(n * m):
 *     - キャッシュ用配列: O(n)
 *     - キュー: 最悪の場合 O(n * m)
 *     - その他定数計算量の変数等: O(1)
 * 
 * ※ n = amount, m = coins.length
 */
class Solution {
    private final int NO_COMBINATION = -1;

    public int coinChange(int[] coins, int amount) {
        int[] amountToMinCoinCount = new int[amount + 1];
        Arrays.fill(amountToMinCoinCount, NO_COMBINATION);
        amountToMinCoinCount[0] = 0;

        for (int currentAmount = 1; currentAmount <= amount; currentAmount++) {
            int minCoinCount = Integer.MAX_VALUE;
            boolean isAnyCombinationFound = false;
            for (int coin : coins) {
                int remainder = currentAmount - coin;
                if (remainder < 0 || amountToMinCoinCount[remainder] == NO_COMBINATION) {
                    continue;
                }

                minCoinCount = Math.min(minCoinCount, amountToMinCoinCount[remainder] + 1);
                isAnyCombinationFound = true;
            }
            if (isAnyCombinationFound) {
                amountToMinCoinCount[currentAmount] = minCoinCount;
            }
        }
        return amountToMinCoinCount[amount];
    }
}
```

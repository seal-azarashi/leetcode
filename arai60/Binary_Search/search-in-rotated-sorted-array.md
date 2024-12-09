# 33. Search in Rotated Sorted Array

LeetCode URL: https://leetcode.com/problems/search-in-rotated-sorted-array/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

次のように考えながら解きました

- 単純に左から順に target を探していけば見つけることはできる
- ただ、配列は rotated である可能性はあるものの、ソート済なのでもっと効率よく探索が出来そうだ
- ソート済配列の効率が良い探索方法としては二分探索が真っ先に思い浮かぶので、これが用いれるかを検討する
    - (💭 振り返ると、Arai60 のカテゴリや問題文に log n で解けという文面があったのを見ていたので、思いつくのは当然ですね)
    - (💭 上記のような事前情報なしに思いつけるように経験値を積んでいきたいところです)
- 問題に書かれている次の例を考える: nums = `[4,5,6,7,0,1,2]`, target = 0
    - 中央 (index: 3) の値を見たとき、target がそれよりも右にあるか左にあるかを判別できるだろうか？
    - `[4,5,6,7,0,1,2]` を眺めれば、target が右にあることは分かる。これはなぜか？
        - もちろん全部見えているからというのもあるが、全部見えなくても分かるような気はする
    - 実際はこれしか見えてないとしたらどうか `[ , , ,7, , , ]`
        - これだと分からない
    - 二分探索では left, right のインデックスを用いるが、これらが両端の値を指しているとして、それも見えていたらどうか？
    - このように見えている `[4, , ,7, , ,2]`
        - これなら target (0) は右にあるはずと分かる
        - なぜ分かるかというと、4から7の間に0は入り得ないからである
        - また、7から2の間には0は入り得る
- もう一つ、target が左にあると判定される例を考える: : nums = `[7,0,1,2,4,5,6]`, target = 0
    - 最初の探索でこのように見えている: `[7, , ,2, , ,6]`
        - これなら target (0) は左にあるはずと分かる
        - なぜ分かるかというと、2から6の間に0は入り得ないからである
        - また、7から2の間には0は入り得る
- 両方のケースを考えると、ソートされている範囲に着目し、その中に target が含まれる場合はその範囲を、含まれない場合はその範囲外を見るようにすればよさそうだ
- どう処理を書くかを考える
    - 二分探索のため、またソート済配列の検索のため、配列の右端、左端をそれぞれ指す left, right を宣言する必要がある
    - これを while 文で条件を満たさなくなるまでループさせ、ループが終わるのをもって target が発見されなかったこととする
        - ループ条件は何であるべきか
        - `left < right` の場合
            - 検索対象を一つに絞り込んだ or 配列に要素が一つしかない時に値を発見できなくなるのでだめ
        - 上記の問題を回避するため `left <= right` とする必要があるだろうか
    - 各ループではイテレーションの最初に中央の値を指す index (middle とする) を算出する必要がある
        - nums.length の最大値は 5000 なので `(left + right) / 2` で問題ないはず
            - 桁溢れする可能性があるなら `left + (right - left) / 2` としたい
    - ソートされている範囲がどこかを判定するため、nums[left] と nums[middle] を比較する
        - nums[left] が nums[middle] よりも大きい場合: ソートされているのは middle から右側となる
        - nums[left] が nums[middle] よりも小さい場合: ソートされているのは middle から左側となる
        - (💭 nums[middle] と nums[right] を比較しても良く、特にこだわりはないです)
    - ソートされている範囲に target が含まれるかどうかを見て、含まれるならその範囲内をさらに見るように、含まれていないならその範囲外を見るように left もしくは right の値を更新する
- ここまで考えてコードを書き始めた
- コードを書いている途中に考えたことについてはコードコメントを参照

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int middle = (left + right) / 2;

            // 💭 middle が target を指しているかどうかの判定をどこで行うかについて考えてませんでした
            // 💭 後続処理で考えることが少なくなることを期待し、middle を算出したらすぐに判定するようここに書きました
            if (nums[middle] == target) {
                return middle;
            }

            // 💭 ソートされているのが (middle から) 左側だと示したくてこの変数名にしました
            // 💭 left と middle が同じ index を指す (左側の配列の要素が一つしか無いとき) ケースを考慮し比較演算子は `<=` にしました
            boolean isLeftSideSorted = nums[left] <= nums[middle];
            if (isLeftSideSorted) {
                // 💭 middle の位置に target が無いことは確定しているので `nums[left] <= target && target < nums[middle]` としてもよかったのですが
                //    パズル感が出るのでこのようにしました
                if (nums[left] <= target && target <= nums[middle]) {
                    // 💭 middle の位置に target は無いことが確定しているので `right = middle - 1` としてもいいですが、それもパズル感に繋がりそうなのでこのようにしています
                    right = middle;
                } else {
                    left = middle + 1;
                }
            } else {
                if (nums[middle] <= target && target <= nums[right]) {
                    left = middle;
                } else {
                    right = middle - 1;
                }
            }
        }
        return -1;
    }
}
```

ここまで書き上げて次のことに気づきました

- `-1` がマジックナンバー化していてよくない
- `1 <= nums.length <= 5000` は Leetcode の constraints でしかないので、やはり middle の算出は桁溢れする可能性を考慮した方が良さそう

上記踏まえて次のように修正しました

```java
class Solution {
    private static final int NOT_FOUND = -1;

    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            }

            boolean isLeftSideSorted = nums[left] <= nums[middle];
            if (isLeftSideSorted) {
                if (nums[left] <= target && target <= nums[middle]) {
                    right = middle;
                } else {
                    left = middle + 1;
                }
            } else {
                if (nums[middle] <= target && target <= nums[right]) {
                    left = middle;
                } else {
                    right = middle - 1;
                }
            }
        }
        return NOT_FOUND;
    }
}
```

計算量は両者ともに TC: O(log n), SC: O(1)。n は nums.length。

その他感想を書いておきます

- もっと早くコードを書き始めて良かった気がする
    - while 文の条件とかちょっと考えすぎでは？
    - 実際は紙に書きながら考えており、pseudo code は既に書き始めてはいたが
- 別に middle を算出したらすぐに target の判定をしても後で考えることが減るということはなかった
    - 結局 middle が target を指さないことを前提とするかどうか、後の条件文を書く際に考えていた
- 考えを書いていて、ソート済配列の中に歯抜けの値がある (ex. `[1,3,4]` のような場合) ことを意識せずに考えていたことに気づいた
    - 今回はたまたま困らなかったが、問題の最初にちゃんと認識しておかないと問題によっては詰まりそう

## Step 2

### 値に不正がある場合の考慮

「ゴミデータつっこまれると無限ループになります?」と oda さんの指摘 ([こちらの9番](https://discord.com/channels/1084280443945353267/1196472827457589338/1196472926862577745))にあったので、上記の実装についてはどうか考えてみました。

- 配列がソートされていない場合:
    - 例えば nums: `[3, 1, 1, 1, 1, 4, 2]`, target: 4 の場合
    - これはもちろん機能せず、-1 が返る
- 値に重複が含まれる場合:
    - 例えば nums: `[3, 1, 1, 1, 1, 1, 2]` のような場合
    - target が 2, 3 の場合ちゃんと機能する
    - 複数ある 1 が target だった場合、最初に middle の値になるどれかが返る

値が重複していてもソートが守られていればちゃんと target を指す index が返ると考えて良さそうに思います。  
無限ループになることもなさそうです。

一点注意として、これについては Claude に手伝って自前で次のテストケースを用意して検証したものになります。  
何か観点の漏れ、認識間違いがありましたらご指摘頂けますと嬉しいです🙇‍♂️  

```java
int[][] testCases = {
    {1, 1, 1, 1, 1, 1, 1},  // すべて同じ値
    {1, 1, 1, 2, 1, 1, 1},  // 1つだけ異なる値
    {2, 2, 2, 1, 2, 2, 2},  // 中央に最小値
    {1, 1, 1, 1, 2, 1},     // 末尾付近に異なる値
    {3, 1, 1, 1, 1, 1, 2},  // 先頭に異なる値
    {3, 1, 1, 1, 1, 4, 2},
    {1},                    // 要素が1つ
    {2, 1},                 // 要素が2つ
};
```

### 最小値を見つけてそこで左右に分割して考えるパターン

[他の方が書かれていた](https://discord.com/channels/1084280443945353267/1196472827457589338/1196472926862577745)のを参考に書いてみました。  
こちらもスラスラ書けるようにしておきたいです。

```java
class Solution {
    public int search(int[] nums, int target) {
        int pivot = findPivotInRotatedSortedArray(nums);
        int from = 0, to = nums.length;
        if (nums[pivot] <= target && target <= nums[nums.length - 1]) {
            from = pivot;
        } else {
            to = pivot;
        }
        int foundIndex = Arrays.binarySearch(nums, from, to, target);
        if (foundIndex >= 0) {
            return foundIndex;
        }
        return -1;
    }

    private int findPivotInRotatedSortedArray(int[] nums) {
        int left = 0; // これより左には最小値がないことを示す
        int right = nums.length - 1; // 最小値かそれよりも右を指す
        while (left < right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] <= nums[right]) {
                right = middle;
            } else {
                left = middle + 1;
            }
        }
        return right;
    }
}
```

## Step 3

```java
class Solution {
    private final int NOT_FOUND = -1;

    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            }

            if (nums[left] <= nums[middle]) { // middle から左の区間がソートされている場合
                if (nums[left] <= target && target < nums[middle]) { // middle から左の区間に target があると思われる場合
                    right = middle;
                } else {
                    left = middle + 1;
                }
            } else {
                if (nums[middle] <= target && target <= nums[right]) {
                    left = middle;
                } else {
                    right = middle - 1;
                }
            }
        }
        return NOT_FOUND;
    }
}
```

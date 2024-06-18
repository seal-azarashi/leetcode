# 20. Valid Parentheses

LeetCode URL: https://leetcode.com/problems/kth-largest-element-in-a-stream/

この問題は Java で解いています。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 4 m 34 s
class KthLargest {
    int k;
    PriorityQueue<Integer> kthLargestElements = new PriorityQueue<>();

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int num : nums) {
            kthLargestElements.add(num);
            if (kthLargestElements.size() > k) kthLargestElements.poll();
        }
    }
    
    public int add(int val) {
        kthLargestElements.add(val);
        if (kthLargestElements.size() > k) kthLargestElements.poll();
        return kthLargestElements.peek();
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- Arai 60 の問題カテゴリに PriorityQueue と書いてあったのでそれを使うものだと分かってしまう
- Java の PriorityQueue クラスの存在を知っていたので、それを使った次の解法を思いつく:
    1. クラスメンバーに k と kthLargestElements (priority queue のインスタンス) を登録
    2. KthLargest では:
        - 引数 k を対応するクラスメンバーに代入
        - nums については、数値の最も大きい k 個の要素だけ残し、その中で最も小さい値を参照出来るよう、その各要素を kthLargestElements に代入後、要素数 k を超過した際に先頭要素を削除して、を全要素を対象に繰り返す
    3. add では同じ KthLargest で nums の各要素に対して行ったのと同じ手順で kthLargestElements への値の追加/削除を行い、最後にその先頭要素を返す
- 実装後一回でパスした
- メソッド内でクラスメンバーを扱う際に `this.` を明示的につけるべきか悩んだが、今回は全変数の数もクラス全体の行数も少ないので、必要な箇所以外にはつけないことにした (必要な箇所: `this.k = k;`)

## Step 2

今回このステップでは、Step 1 に対して行った実装を修正するといったことはしませんでした。最終的にそうするよう判断した経緯は次の通りです。

- LeetCode の上位解法5つを見たところ、現実的に取り入れられそうなアイデアがひとつあった
- そのアイデアは、KthLargest と add で処理が重複する部分、具体的には kthLargestElements への挿入/削除の処理を共通化するというもの
- Step 1 のコードに上記アイデアをアプライすると次のようになる

```java
class KthLargest {
    int k;
    PriorityQueue<Integer> kthLargestElements = new PriorityQueue<>();

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int num : nums) add(num); // 👈 修正箇所
    }
    
    public int add(int val) {
        kthLargestElements.add(val);
        if (kthLargestElements.size() > this.k) kthLargestElements.poll();
        return kthLargestElements.peek();
    }
}
```

- 修正内容は、kthLargestElements への挿入/削除のため、KthLargest から add を呼び出すというもの
- 実際自分がこの解法を見たときも思ったが、元のコードよりも認知不可が大きいと感じる
- この実装だと、KthLargest の実装を理解するために、その下の行にある、別の用途で使うと問題文に書いてあった add の中身をまず把握する必要がある
- さらには add が返す値を KthLargest で呼び出す際には無視しても良いと読み取らなければならない
    - 自分がこの実装のレビュワーだとしたら、本当に無視して大丈夫なのかを考えると思い、これも負担に感じた
- そもそも返り値のある関数を、それを無視して使うというのは例外的な使い方のように感じる
- つまるところ、たった2行の処理を共通化するためだけに、可読性を下げる結果になってしまうと感じ、採用を見送った

## Step 3

約1分半で3回パスした。

## Step 4

### 解法の列挙

Step 1, 2 で実施出来ていなかった解法のリストアップを試みます。

#### ソート済リストの使用

リストがソートされている状態を保ちながら要素を追加し、k 番目の要素を返す。

- 動的リスト (今回は Java を使うため Array List) を使う
- 要素の追加手順 O(mn):
    - binary search で挿入位置を特定 (O(log n))
    - 指定した位置に挿入 (O(n))
    - 上記を nums の要素数 m だけ実施する
- binary search には [Collections.binariSearch](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#binarySearch-java.util.List-T-) を使用
    - 以下公式ドキュメント読んで面白いと思ったポイントのメモ:
        - ちなみに検索対象の値がない場合、返り値は (-(検索対象の値より大きい最初の要素のインデックス) - 1)
        - 負の値にされる理由は、検索対象が見つからなかったことを表すため
        - 1減算されている理由は、検索対象の値より大きい最初の要素のインデックスが 0 だった場合でも負の値にするため
- 各イテレーションにおける処理の手順は次の通り:
    1. Collections.binariSearch を用いて挿入箇所を特定
        - 返り値が正の値だった場合は返り値をそのまま挿入箇所として使う
        - 返り値が負の値だった場合の挿入箇所: -(返り値) - 1
    2. 挿入する

尚、要素の追加方法として、末尾に値を挿入した後 Array List の sort メソッドを呼ぶ実装も考えたが、これは計算量がより多くなるため好ましくない。

#### 最小ヒープ (priority queue) の使用

Step 1, 2 で実装。

#### 平衡二分探索木

[kazukiii さんのプルリクエスト](https://github.com/kazukiii/leetcode/pull/9/files)で候補に挙がっていた。
Java の標準ライブラリには要素の重複を許容する平衡二分探索木が無いため、[Guava の TreeMultiSet](https://guava.dev/releases/19.0/api/docs/com/google/common/collect/TreeMultiset.html) を使う等して実装するのが良さそう。
TreeMap を使い、value に要素数を保持することで実装することも出来るか。

#### Fenwick tree 上の二分探索

[kazukiii さんのプルリクエスト](https://github.com/kazukiii/leetcode/pull/9/files)で知った解法。
Fenwick tree (Binary Indexed Tree) は要素の更新と区間の和を行うクエリを高速に処理できるデータ構造。Java の標準ライブラリになく、またこの問題を解くために実装するのは too much ではあるが、こういったものがあるというのを知れて勉強になった。

### Step 1 の解法の効率化

oda さんからのレビューを受け、Step 1 で書いた処理の効率化を行った。
元の実装では要素数が k 以上になった後、add メソッドが呼ばれる度必ず PriorityQueue の add と poll メソッドが呼ばれていたが、val が先頭要素よりも小さい場合これを実行しないようにした。

```java
class KthLargest {
    int k;
    PriorityQueue<Integer> kthLargestElements = new PriorityQueue<>();

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int num : nums) add(num);
    }
    
    public int add(int val) {
        if (kthLargestElements.size() >= this.k && val < kthLargestElements.peek()) return kthLargestElements.peek();

        kthLargestElements.add(val);
        if (kthLargestElements.size() > k) kthLargestElements.poll();
        return kthLargestElements.peek();
    }
}
```

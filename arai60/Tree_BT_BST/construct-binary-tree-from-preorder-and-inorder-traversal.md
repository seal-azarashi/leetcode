# 105. Construct Binary Tree from Preorder and Inorder Traversal

LeetCode URL: https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

解答が思いつかなかったのでこちらの動画の解法を見ました: https://youtu.be/ihj4IQGZ2zc?si=ag5X1J1bub7O2uXr  
内容は次のように理解しています:
- 前提知識:
    1. Preorder traversal で得られる配列の先頭要素はルートノードである
    2. Inorder traversal で得られる配列において、 1 と同じ要素より左および右にある要素群は、それぞれ左、右のツリーを構成する
    3. Preorder traversal で得られる配列において、先頭要素以降の集合は、あるインデックスを境に左右の子ツリーに分かれる
- イテレーションごとに次の操作を実施すればツリーが生成できる:
    1. 配列 preorder の先頭要素を用いてノードを作る
    2. 配列 inorder から、 1 と同じ要素が格納される index を見つける
    3. 2 のインデックスを用いて preorder, inorder を分割し、左右の子ノードを生成するためそれぞれ次のイテレーションに渡す
        - 2のインデックスは左子ツリーの要素数と対応する
        - preorder は 先頭要素がルートノードなので、それを含めない [1:2のインデックス+1] の範囲を左、[2のインデックス+1:preorder.length] の範囲を右の子ツリーとする
        - inorder は [0:2のインデックス] の範囲が左、[2のインデックス+1:inorder.length] の範囲が右の子ツリーとなる
        - ※上記どちらも閉開区間での範囲指定

上記理解した上で、次のようなことを考えながら実装していました:

- Complete binary tree でないかもしれないのでスタックオーバーフローが起こる懸念があるが、現時点の自分の理解度だとループを用いた実装を綺麗に書ける自信がないので、問題文の constraints が守られることを前提に再帰処理で書いてみよう
- 引数について、 null でないことやそれぞれ要素数が等しいといった前提で処理を書いているが、全部コメントに残すと見た目が煩雑になりそうなのでひとまず書かないでおこう
- 配列の分割には Arrays.copyOfRange() を使うのが良さそうだ
    - 指定するインデックスをどうするか悩んでいたところ、 copyOfRange() のドキュメントに次の文言を見つけた: The initial index of the range (from) must lie between zero and original.
    - 存在しない要素のインデックスであっても配列の length を超えないのであればエラーにならないので、綺麗に書けそう

```java
// 時間計算量: O(n^2):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n^2): 各イテレーションで、新規配列を作成するため、引数に渡された配列の合計要素数 (配列の要素数 - 1) * 2 と同じ回数のコピー処理を行う
// 空間計算量: O(n): 配列が複数作成され、それら要素数の合計は引数に渡される配列たちの合計要素数より少ない
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if (preorder.length == 0 || inorder.length == 0) {
            return null;
        }

        TreeNode node = new TreeNode(preorder[0]);
        int inorderMiddle = 0;
        for (int i = 0; i < preorder.length; i++) {
            if (preorder[0] == inorder[i]) {
                inorderMiddle = i;
            }
        }
        node.left = buildTree(
            Arrays.copyOfRange(preorder, 1, inorderMiddle + 1),
            Arrays.copyOfRange(inorder, 0, inorderMiddle)
        );
        node.right = buildTree(
            Arrays.copyOfRange(preorder, inorderMiddle + 1, preorder.length),
            Arrays.copyOfRange(inorder, inorderMiddle + 1, inorder.length)
        );
        return node;
    }
}
```

## Step 2

### 配列の部分コピーを使用 (Step 1 改良版)

- preorder と inorder の長さは常に等しいはずなので、 base case の条件を、片方だけ確認するように修正

```java
// 時間計算量: O(n^2):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n^2): 各イテレーションで、新規配列を作成するため、引数に渡された配列の合計要素数 (配列の要素数 - 1) * 2 と同じ回数のコピー処理を行う
// 空間計算量: O(n): 配列が複数作成され、それら要素数の合計は引数に渡される配列たちの合計要素数より少ない
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        // The length of preorder and inorder should be the same
        if (preorder.length == 0) {
            return null;
        }

        TreeNode node = new TreeNode(preorder[0]);
        int inorderMiddle = 0;
        for (int i = 0; i < preorder.length; i++) {
            if (preorder[0] == inorder[i]) {
                inorderMiddle = i;
            }
        }
        node.left = buildTree(
            Arrays.copyOfRange(preorder, 1, inorderMiddle + 1),
            Arrays.copyOfRange(inorder, 0, inorderMiddle)
        );
        node.right = buildTree(
            Arrays.copyOfRange(preorder, inorderMiddle + 1, preorder.length),
            Arrays.copyOfRange(inorder, inorderMiddle + 1, inorder.length)
        );
        return node;
    }
}
```

### インデックスで走査範囲を指定

- 配列のコピーを作成する代わりにインデックスを使うことで計算量を削減する
- イテレーションごとに inorder の要素を線形探索せずに済むように map を使っている
- preorderIndex は 1 ずつ increment すればいいことが分かってれば理解しやすい印象

```java
// 時間計算量: O(n):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n): inorder の値とインデックスのマップを作成
//     - O(1): イテレーションごとに実行されるマップへのアクセス
// 空間計算量: O(n):
//     - O(n): 引数に渡される配列の合計要素と同じ数のツリーを生成
//     - O(n): inorder の値とインデックスのマップを作成
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        // Make it an array so that it could behave as reference type value
        int[] preorderIndex = new int[]{ 0 };
        Map<Integer, Integer> valToInorderIndex = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) {
            valToInorderIndex.put(inorder[i], i);
        }
        return buildTree(preorder, 0, inorder.length, preorderIndex, valToInorderIndex);
    }
    
    private TreeNode buildTree(int[] preorder, int left, int right, int[] preorderIndex, Map<Integer, Integer> valToInorderIndex) {
        if (left == right) {
            return null;
        }
        
        int val = preorder[preorderIndex[0]];
        TreeNode node = new TreeNode(val);
        int inorderIndex = valToInorderIndex.get(val);

        preorderIndex[0]++;

        node.left = buildTree(preorder, left, inorderIndex, preorderIndex, valToInorderIndex);
        node.right = buildTree(preorder, inorderIndex + 1, right, preorderIndex, valToInorderIndex);
        return node;
    }
}
```

- 可読性向上のためにメンバー変数を使ったパターンも書いてみる
- public メソッドの buildTree() は何度呼ばれても同じ結果を返すために初期化処理を入れた
- 【🚨ご意見ください】ここでメンバー変数を使うことについて、ネガティブな意見があれば教えてもらえますと嬉しいです
    - メンバー変数はクラスから生成されるオブジェクトが保持する状態のようなものだと認識している
    - しかしここではあくまで buildTree() の実装を見やすくする用途で用いられているのが懸念
    - 賛否両論ありそうなので意見を聞いてみたい

```java
// 時間計算量: O(n):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n): inorder の値とインデックスのマップを作成
//     - O(1): イテレーションごとに実行されるマップへのアクセス
// 空間計算量: O(n):
//     - O(n): 引数に渡される配列の合計要素と同じ数のツリーを生成
//     - O(n): inorder の値とインデックスのマップを作成
class Solution {
    private int preorderIndex;
    private Map<Integer, Integer> valToInorderIndex;

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorderIndex = 0;
        this.valToInorderIndex = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) {
            this.valToInorderIndex.put(inorder[i], i);
        }
        return buildTreeRecursively(preorder, 0, inorder.length);
    }
    
    private TreeNode buildTreeRecursively(int[] preorder, int left, int right) {
        if (left == right) {
            return null;
        }
        
        int val = preorder[this.preorderIndex];
        TreeNode node = new TreeNode(val);
        int inorderIndex = this.valToInorderIndex.get(val);

        this.preorderIndex++;

        node.left = buildTreeRecursively(preorder, left, inorderIndex);
        node.right = buildTreeRecursively(preorder, inorderIndex + 1, right);
        return node;
    }
}
```

### スタックを用いた実装

TODO

### その他気になったコメントなど

- > 「自分で手作業でできる」ようにして「人に手作業でできるように説明をする」というプロセスを踏むと、明らかに無駄なところは気がつくように思うんですよね: https://github.com/goto-untrapped/Arai60/pull/53/files#r1780608187
    - なるほどたしかに

## Step 3



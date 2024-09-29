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
- イテレーションごとに次の操作を実施すればツリーが生成できる:
    1. 配列 preorder の先頭要素を用いてノードを作る
    2. 配列 inorder から、 1 と同じ要素が格納される index を見つける
    3. 2 のインデックスを用いて preorder, inorder を分割し、左右の子ノードを生成するためそれぞれ次のイテレーションに渡す

```java
// 時間計算量: O(n^2):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n^2): 各イテレーションで、新規配列を作成するため、引数に渡された配列の合計要素数 (配列の要素数 - 1) * 2 と同じ回数のコピー処理を行う
// 空間計算量: O(n): 配列が複数作成され、それら要素数の合計は引数に渡される配列たちの合計要素数と等しい
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

次のようなことを考えながら実装していました:

- Complete binary tree でないかもしれないのでスタックオーバーフローが起こる懸念があるが、現時点の自分の理解度だとループを用いた実装を綺麗に書ける自信がないので、問題文の constraints が守られることを前提に再帰処理で書いてみよう
- 引数について、 null でないことやそれぞれ要素数が等しいといった前提で処理を書いているが、全部コメントに残すと見た目が煩雑になりそうなのでひとまず書かないでおこう
- 配列の分割には Arrays.copyOfRange() を使うのが良さそうだ
    - 指定するインデックスをどうするか悩んでいたところ、 copyOfRange() のドキュメントに次の文言を見つけた: The initial index of the range (from) must lie between zero and original.
    - 存在しない要素のインデックスであっても配列の length を超えないのであればエラーにならないので、綺麗に書けそう

## Step 2

- 計算量は O(n^2) かも: https://github.com/Mike0121/LeetCode/pull/12#discussion_r1725020486

### 配列の部分コピーを使用 (Step 1 改良版)

- preorder と inorder の長さは常に等しいはずなので、 base case の条件を、片方だけ確認するように修正

```java
// 時間計算量: O(n^2):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n^2): 各イテレーションで、新規配列を作成するため、引数に渡された配列の合計要素数 (配列の要素数 - 1) * 2 と同じ回数のコピー処理を行う
// 空間計算量: O(n): 配列が複数作成され、それら要素数の合計は引数に渡される配列たちの合計要素数と等しい
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
- preorder の特徴から？ preorderIndex は 1 ずつ increment すればいいことが分かってればわかりやすい
- レビュワーの方に、プライベート変数を使うのは評価としてどうなのか聞いてみたい
    - 個人的には、クラスを拡張する際に意図しない使い方がされてしまわないか心配
    - そもそも leetcode が用意した関数内でしか使わないので、スコープは狭いほうがいい
    - けど int 型で良いものを配列にするといった気持ち悪いことをしなければならないケースがあるのが気になる

```java
// 時間計算量: O(n):
//     - O(n): ノード生成処理を引数に渡される配列の要素数と同じ回数実行
//     - O(n): inorder の値とインデックスのマップを作成
//     - O(n): イテレーションごとにマップにアクセス
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

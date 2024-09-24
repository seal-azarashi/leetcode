# 98. Validate Binary Search Tree

LeetCode URL: https://leetcode.com/problems/validate-binary-search-tree/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): Balanced binary tree とは限らないので、最大で全ノードがスタックに格納される
class Solution {
    private record NodeMinMax(TreeNode node, long min, long max) {};

    public boolean isValidBST(TreeNode root) {
        if (root == null) {
            return false;
        }

        Deque<NodeMinMax> nodeStack = new ArrayDeque<>();
        nodeStack.push(new NodeMinMax(root, Long.MIN_VALUE, Long.MAX_VALUE));
        while (!nodeStack.isEmpty()) {
            NodeMinMax nodeMinMax = nodeStack.pop();
            TreeNode node = nodeMinMax.node;
            long min = nodeMinMax.min, max = nodeMinMax.max;
            if (node == null) {
                continue;
            }
            boolean isValid = min < node.val && node.val < max;
            if (!isValid) {
                return false;
            }
            nodeStack.push(new NodeMinMax(node.left, min, node.val));
            nodeStack.push(new NodeMinMax(node.right, node.val, max));
        }
        return true;
    }
}
```

次のようなことを考えながら実装していました:

- Balanced binary tree とは限らないので、再帰関数として実装すると最大で 10^4 要素がスタックに入る可能性があり、スタックオーバーフローの懸念があると判断
- なのでスタックを使って実装する
- 最大値、最小値が Java の integer の MIN_VALUE, MAX_VALUE なので、工夫が必要
    - すでに最大、最小値が現れたかどうかを boolean 値に格納するといった事をすれば integer でも扱えそうではある
    - 今回は処理の読みやすさをとって long 値を使うことにする
        - サイズが integer の2倍しかないのでクリティカルな要因にはならないはず

## Step 2

### 再帰による実装

練習で再帰の実装も書く。

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): Balanced binary tree とは限らないので、最大で全ノードの数だけスタックフレームが生成される
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValidBSTRecursively(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private boolean isValidBSTRecursively(TreeNode node, long min, long max) {
        if (node == null) {
            return true;
        }
        boolean isValid = min < node.val && node.val < max;
        if (!isValid) {
            return false;
        }
        return
            isValidBSTRecursively(node.left, min, node.val) &&
            isValidBSTRecursively(node.right, node.val, max);
    }
}
```

### In-order traversal (スタック)

自分では思いつかなかったが他の方が書かれていたこちらも実装してみた。特に読みやすく感じた [ahayashi さんの実装](https://github.com/hayashi-ay/leetcode/pull/38)をおおいに参考にしています。  
Valid なツリーに対して in-order traversal をすると non-decreasing order にソートされた値が得られることが分かっていればすんなり頭に入ってくるのでこっちの方が好みかも。

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): Balanced binary tree とは限らないので、最大で全ノードがスタックに格納される
class Solution {
    public boolean isValidBST(TreeNode root) {
        Deque<TreeNode> nodeStack = new ArrayDeque<>();
        TreeNode node = root;
        long previousVal = Long.MIN_VALUE;
        while (true) {
            while (node != null) {
                nodeStack.push(node);
                node = node.left;
            }
            if (nodeStack.isEmpty()) {
                break;
            }
            node = nodeStack.pop();
            boolean isValid = previousVal < node.val;
            if (!isValid) {
                return false;
            }
            previousVal = node.val;
            node = node.right;
        }
        return true;
    }
}
```

### その他感想など

- nittoco さんの思考ログの書き方がとても良くて参考になった: https://github.com/nittoco/leetcode/pull/35/files
    - これだけしっかり書いてあればレビュワーの方々もコメントしやすい
    - たくさんレビューがついてそれについて考える時間は自分の理解をとても深めてくれそう
    - 最終的な実装に至るまでに書いた古いバージョンについても今後は残しておいて思考ログを書けるようにしておこう
- BFS でも書いたが、 (引数に何が来るのか予測出来ない限り) DFS と計算量に違いはない上、実装もスタックをキューに変えるだけで既存実装とほぼ変わらないので割愛
- TODO: 常識範囲外とのことで一旦飛ばした Morris in-order を Arai 60 が一通り終わったら見てみる: https://github.com/nittoco/leetcode/pull/35/files

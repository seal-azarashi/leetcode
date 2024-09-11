# 617. Merge Two Binary Trees

LeetCode URL: https://leetcode.com/problems/merge-two-binary-trees/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 10分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) {
            return root2;
        }
        if (root2 == null) {
            return root1;
        }

        root1.val = root1.val + root2.val;
        root1.left = mergeTrees(root1.left, root2.left);
        root1.right = mergeTrees(root1.right, root2.right);
        return root1;
    }
}
```

次のようなことを考えながら実装していました:

- 再帰もしくはスタックを使った実装になりそう
- ペアとなるノードの合算値が取りうる範囲は `-2*10^4` ~ `2*10^4` なので、 Java の整数値 (`-2^31` ~ `2^31-1`) なら問題なく扱える
- スタックオーバーフローになるリスクを避けるためにスタックを使いたいが、綺麗なロジックが思いつかない
- constraints にノード数が最大2000とあるので、スタックオーバーフローにはならないと判断しひとまず再帰処理を実装することにする
- 結構簡潔に書けた
- 書けたものの、引数 root1 に修正を加える実装となったのが気になっている
    - merge するのが目的なので引数の値が変わっても問題は無いかも知れないが、あるかもしれない
    - 面接ではちゃんと面接官に伝えて合意を取りたい
    - 引数に修正を加えてはならないと判断される場合、引数の deep copy を作るように追加実装が必要かなと考えた

## Step 2

### スタックを使った実装

```java
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) {
            return root2;
        }

        Deque<TreeNode[]> treeNodePairStack = new ArrayDeque<>();
        treeNodePairStack.push(new TreeNode[] { root1, root2 });
        while (!treeNodePairStack.isEmpty()) {
            TreeNode[] pair = treeNodePairStack.pop();
            if (pair[1] == null) {
                continue;
            }

            pair[0].val += pair[1].val;
            if (pair[0].left == null) {
                pair[0].left = pair[1].left;
            } else {
                treeNodePairStack.push(new TreeNode[] { pair[0].left, pair[1].left });
            }
            if (pair[0].right == null) {
                pair[0].right = pair[1].right;
            } else {
                treeNodePairStack.push(new TreeNode[] { pair[0].right, pair[1].right });
            }
        }

        return root1;
    }
}
```

- ちゃんと書けなかったので Leetcode の editorial を参考にした
    - root1 側に属するノードに常に値が入っていることを前提に、スタックにノードのペアを積み、処理するような実装
    - 各イテレーションではroot2 側を (null でなければ) root1 にマージしていく
- 値に対する修正が必要なので Record が使えないため可読性は低い印象 (Record の値は immutable)
- これもまた引数 root1 に修正を加える実装となったのが気になっている

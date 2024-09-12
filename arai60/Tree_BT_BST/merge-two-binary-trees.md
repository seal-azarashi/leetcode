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
    - 引数に修正を加えてはならないと判断される場合、引数の deep copy を作るように追加実装が必要かなと考えた (Java にはないので)

## Step 2

### スタックを使った実装

```java
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) {
            return root2;
        }

        Deque<TreeNode[]> treeNodePairStack = new ArrayDeque<>();
        treeNodePairStack.push(new TreeNode[]{ root1, root2 });
        while (!treeNodePairStack.isEmpty()) {
            TreeNode[] pair = treeNodePairStack.pop();
            TreeNode node1 = pair[0], node2 = pair[1];
            if (node2 == null) {
                continue;
            }

            node1.val += node2.val;
            if (node1.left == null) {
                node1.left = node2.left;
            } else {
                treeNodePairStack.push(new TreeNode[]{ node1.left, node2.left });
            }
            if (node1.right == null) {
                node1.right = node2.right;
            } else {
                treeNodePairStack.push(new TreeNode[]{ node1.right, node2.right });
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

### 非破壊的実装

やはり引数に修正を加える (破壊的な) 動作の懸念点ついて別の方のプルリクでレビューがされていたので、非破壊的な実装も書いてみます。

まず再帰を用いた実装から。

```java
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

        TreeNode newNode = new TreeNode(root1.val + root2.val);
        newNode.left = mergeTrees(root1.left, root2.left);
        newNode.right = mergeTrees(root1.right, root2.right);
        return newNode;
    }
}
```

次にスタックを用いた実装。マージ済みのノードを対象に、イテレーションごとにその子ノードを生成していくような処理になっています。

```java
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
        
        TreeNode mergedRoot = new TreeNode(root1.val + root2.val);
        Deque<TreeNode[]> treeNodeStack = new ArrayDeque<>();
        treeNodeStack.push(new TreeNode[]{ root1, root2, mergedRoot });
        while (!treeNodeStack.isEmpty()) {
            TreeNode[] treeNodes = treeNodeStack.pop();
            TreeNode node1 = treeNodes[0], node2 = treeNodes[1], mergedNode = treeNodes[2];
            
            if (node1.left != null || node2.left != null) {
                if (node1.left != null && node2.left != null) {
                    mergedNode.left = new TreeNode(node1.left.val + node2.left.val);
                    treeNodeStack.push(new TreeNode[]{ node1.left, node2.left, mergedNode.left });
                } else if (node1.left != null) {
                    mergedNode.left = new TreeNode(node1.left.val);
                    treeNodeStack.push(new TreeNode[]{ node1.left, new TreeNode(0), mergedNode.left });
                } else {
                    mergedNode.left = new TreeNode(node2.left.val);
                    treeNodeStack.push(new TreeNode[]{ new TreeNode(0), node2.left, mergedNode.left });
                }
            }
            
            if (node1.right != null || node2.right != null) {
                if (node1.right != null && node2.right != null) {
                    mergedNode.right = new TreeNode(node1.right.val + node2.right.val);
                    treeNodeStack.push(new TreeNode[]{ node1.right, node2.right, mergedNode.right });
                } else if (node1.right != null) {
                    mergedNode.right = new TreeNode(node1.right.val);
                    treeNodeStack.push(new TreeNode[]{ node1.right, new TreeNode(0), mergedNode.right });
                } else {
                    mergedNode.right = new TreeNode(node2.right.val);
                    treeNodeStack.push(new TreeNode[]{ new TreeNode(0), node2.right, mergedNode.right });
                }
            }
        }
        
        return mergedRoot;
    }
}
```

これで破壊的な動作はなくなりましたが、どちらも root1, root2 をそのまま返す可能性があるのが気がかりです。問題文には "You need to merge the two trees into a new binary tree" とあるので、これを満たさないと判断されるようでしたらやはり deep copy を実施する関数を自前で実装するなり、そういった関数がある前提で処理を書くことになるのかなと考えました。

## Step 3

面接では速度が求められるので、比較的シンプルかつ破壊的操作のない再帰処理を用いた実装を選びました。

```java
// 解いた時間: 4分ぐらい
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if (root1 == null) {
            return root2;
        }
        if (root2 == null) {
            return root1;
        }

        TreeNode mergedNode = new TreeNode(root1.val + root2.val);
        mergedNode.left = mergeTrees(root1.left, root2.left);
        mergedNode.right = mergeTrees(root1.right, root2.right);
        return mergedNode;
    }
}
```

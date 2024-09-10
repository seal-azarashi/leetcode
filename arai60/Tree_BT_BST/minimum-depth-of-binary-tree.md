# 111. Minimum Depth of Binary Tree

LeetCode URL: https://leetcode.com/problems/minimum-depth-of-binary-tree/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 5分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    private static final int MIN_DEPTH_NOT_CALCULATED = -1;

    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Queue<TreeNode> treeNodes = new ArrayDeque<>();
        treeNodes.offer(root);
        int depth = 1;
        while (!treeNodes.isEmpty()) {
            int currentLevelNodesCount = treeNodes.size();
            for (int i = 0; i < currentLevelNodesCount; i++) {
                TreeNode node = treeNodes.poll();
                if (node.left == null && node.right == null) {
                    return depth;
                }
                if (node.left != null) {
                    treeNodes.offer(node.left);
                }
                if (node.right != null) {
                    treeNodes.offer(node.right);
                }
            }

            depth++;
        }

        // 処理に異常が無い限りここには到達しません。
        return MIN_DEPTH_NOT_CALCULATED;
    }
}
```

次のようなことを考えながら実装していました:

- Queue を用いた BFS と再帰関数、スタックによる DFS の計3つのアプローチを思いつく
- 1つめのアプローチはスタックオーバーフローのリスクが無く、リーフノードを見つけた時点で処理が終了できて効率がいい、また実装も他2つと比べ理解しやすい内容になると考え、そちらで書いてみた
    - Leetcode の constraints 上はスタックオーバーフローのリスクはほぼ無いと考えらるが、 [oda さんのコメント](https://github.com/kazukiii/leetcode/pull/22/files#r1667746480)にある通り「この環境では大丈夫」なコードはいつ自分の足を撃ち抜いて来るかわからないので避けた
- 次の理由でキュー (ArrayDeque) へは null を挿入していない:
    - そもそも ArrayDeque の各メソッドは null を挿入することを許容しない
    - null チェックをイテレーションごとに都度実施しなくていいなら、そうする越したことはないと思う
    - [ArrayDeque はドキュメント冒頭にある通り早い](https://docs.oracle.com/en%2Fjava%2Fjavase%2F22%2Fdocs%2Fapi%2F%2F/java.base/java/util/ArrayDeque.html)ので、その利点を捨ててまで、例えば null 許容する LinkedList を implementing class にすることもないかなと考える
- minDepth() の最下部は次のようなことを考えて実装した:
    - 処理に異常がない限りは到達しないことを明記
    - Depth の値として期待されない -1 を返す代わりに Exception を throw することも考えたが、後続処理にどのような影響を与えるべきかの議論なしにそこまでやるのは憚られたのでこのようにした
        - 以前同じような議論がされていたので参考まで: https://github.com/seal-azarashi/leetcode/pull/9/files#r1667365546

## Step 2

### スタックによる DFS

```java
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    private record TreeNodeDepth(TreeNode node, int depth) {};

    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Deque<TreeNodeDepth> treeNodeDepths = new ArrayDeque<>();
        treeNodeDepths.push(new TreeNodeDepth(root, 1));
        int minDepth = Integer.MAX_VALUE;
        while (!treeNodeDepths.isEmpty()) {
            TreeNodeDepth nodeDepth = treeNodeDepths.pop();
            if (nodeDepth.node.left == null && nodeDepth.node.right == null) {
                minDepth = Math.min(minDepth, nodeDepth.depth);
            }
            if (nodeDepth.node.left != null) {
                treeNodeDepths.push(new TreeNodeDepth(nodeDepth.node.left, nodeDepth.depth + 1));
            }
            if (nodeDepth.node.right != null) {
                treeNodeDepths.push(new TreeNodeDepth(nodeDepth.node.right, nodeDepth.depth + 1));
            }
        }
        return minDepth;
    }
}
```

### 再帰関数による DFS

```java
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        return traverseMinDepthRecursively(root);
    }

    private int traverseMinDepthRecursively(TreeNode node) {
        if (node.left == null && node.right == null) {
            return 1;
        }

        int leftNodeMinDepth = Integer.MAX_VALUE;
        if (node.left != null) {
            leftNodeMinDepth = traverseMinDepthRecursively(node.left);
        }
        int rightNodeMinDepth = Integer.MAX_VALUE;
        if (node.right != null) {
            rightNodeMinDepth = traverseMinDepthRecursively(node.right);
        }
        return Math.min(leftNodeMinDepth, rightNodeMinDepth) + 1;
    }
}
```

## Step 3

```java
// 解いた時間: 5分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    private static final int MIN_DEPTH_NOT_CALCULATED = -1;
    
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Queue<TreeNode> treeNodes = new ArrayDeque<>();
        treeNodes.offer(root);
        int minDepth = 1;
        while (!treeNodes.isEmpty()) {
            int currentLevelNodeCount = treeNodes.size();
            for (int i = 0; i < currentLevelNodeCount; i++) {
                TreeNode node = treeNodes.poll();
                if (node.left == null && node.right == null) {
                    return minDepth;
                }
                if (node.left != null) {
                    treeNodes.offer(node.left);
                }
                if (node.right != null) {
                    treeNodes.offer(node.right);
                }
            }
            minDepth++;
        }

        return MIN_DEPTH_NOT_CALCULATED;
    }
}
```

# 112. Path Sum

LeetCode URL: https://leetcode.com/problems/path-sum/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 7分ぐらい
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックに入る
class Solution {
    private record NodeWithPathSum(TreeNode node, int pathSum) {};

    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }

        Deque<NodeWithPathSum> nodeStack = new ArrayDeque<>();
        nodeStack.push(new NodeWithPathSum(root, 0));
        while (!nodeStack.isEmpty()) {
            NodeWithPathSum nodeWithPathSum = nodeStack.pop();
            TreeNode node = nodeWithPathSum.node;
            int currentPathSum = node.val + nodeWithPathSum.pathSum;

            boolean isLeafNode = node.left == null && node.right == null;
            if (isLeafNode && currentPathSum == targetSum) {
                return true;
            }

            if (node.right != null) {
                nodeStack.push(new NodeWithPathSum(node.right, currentPathSum));
            }
            if (node.left != null) {
                nodeStack.push(new NodeWithPathSum(node.left, currentPathSum));
            }
        }

        return false;
    }
}
```

次のようなことを考えながら実装していました:

- DFS アプローチで解くのが適切に思える
    - もっと効率よく出来ないか考えるも、全ノードを順にリーフノードまで見ていくのは避けられないと判断
    - 値が0以上しか入り得ないなら途中で打ち切る処理も書けるが、負の数が入り得るのは constraints にも明記されてる
    - 明記されてなくても、面接官に合意を取らない限りは打ち切るような処理は勝手に入れないほうがよさそう
        - 値が0以上であるかどうかについては型システムによって保証されてるわけでもないので
- スタックオーバーフローのリスクを回避するためスタックで実装したい

## Step 2

### 再帰で DFS

スタックオーバーフローのリスクがあるので最適解にはならないと思いますが、一応練習も兼ねて書いてみます。

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックフレームに積まれる
class Solution {
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }
        boolean isLeafNode = root.left == null && root.right == null;
        if (isLeafNode && root.val == targetSum) {
            return true;
        }
        return
            hasPathSum(root.left, targetSum - root.val) ||
            hasPathSum(root.right, targetSum - root.val);
    }
}
```

## Step 3

Step 1 と同じです。

```java
// 解いた時間: 5分ぐらい
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックに入る
class Solution {
    private record NodeWithPathSum(TreeNode node, int pathSum) {};

    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }

        Deque<NodeWithPathSum> nodeStack = new ArrayDeque<>();
        nodeStack.push(new NodeWithPathSum(root, 0));
        while (!nodeStack.isEmpty()) {
            NodeWithPathSum nodeWithPathSum = nodeStack.pop();
            TreeNode node = nodeWithPathSum.node;
            int currentPathSum = node.val + nodeWithPathSum.pathSum;

            boolean isLeafNode = node.left == null && node.right == null;
            if (isLeafNode && currentPathSum == targetSum) {
                return true;
            }

            if (node.right != null) {
                nodeStack.push(new NodeWithPathSum(node.right, currentPathSum));
            }
            if (node.left != null) {
                nodeStack.push(new NodeWithPathSum(node.left, currentPathSum));
            }
        }

        return false;
    }
}
```

## Step 4

### イテレーティブな DFS

次の指摘に対応:

- https://github.com/seal-azarashi/leetcode/pull/24#discussion_r1781228402

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックに入る
class Solution {
    private record NodeWithPathSum(TreeNode node, int pathSum) {};

    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }

        Deque<NodeWithPathSum> nodeStack = new ArrayDeque<>();
        nodeStack.push(new NodeWithPathSum(root, 0));
        while (!nodeStack.isEmpty()) {
            NodeWithPathSum nodeWithPathSum = nodeStack.pop();
            TreeNode node = nodeWithPathSum.node;
            int currentPathSum = node.val + nodeWithPathSum.pathSum;

            if (node.left == null && node.right == null) {
                if (currentPathSum == targetSum) {
                    return true;
                }
                
                continue;
            }

            if (node.right != null) {
                nodeStack.push(new NodeWithPathSum(node.right, currentPathSum));
            }
            if (node.left != null) {
                nodeStack.push(new NodeWithPathSum(node.left, currentPathSum));
            }
        }
        return false;
    }
}
```

スタックには null も入るようにして、チェックを pop してから行うパターン (参考: https://github.com/seal-azarashi/leetcode/pull/24#discussion_r1781230995)

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックに入る
class Solution {
    private record NodeWithPathSum(TreeNode node, int pathSum) {};

    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }

        Deque<NodeWithPathSum> nodeStack = new ArrayDeque<>();
        nodeStack.push(new NodeWithPathSum(root, 0));
        while (!nodeStack.isEmpty()) {
            NodeWithPathSum nodeWithPathSum = nodeStack.pop();
            TreeNode node = nodeWithPathSum.node;

            if (node == null) {
                continue;
            }

            int currentPathSum = node.val + nodeWithPathSum.pathSum;
            if (node.left == null && node.right == null) {
                if (currentPathSum == targetSum) {
                    return true;
                }
                
                continue;
            }
            nodeStack.push(new NodeWithPathSum(node.right, currentPathSum));
            nodeStack.push(new NodeWithPathSum(node.left, currentPathSum));
        }
        return false;
    }
}
```

### 再帰で DFS

```java
// 時間計算量: O(n): 最大で全ノードを走査する
// 空間計算量: O(n): 最大で全ノードがスタックフレームに積まれる
class Solution {
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            return false;
        }
        if (root.left == null && root.right == null) {
            return root.val == targetSum;
        }
        return
            hasPathSum(root.left, targetSum - root.val) ||
            hasPathSum(root.right, targetSum - root.val);
    }
}
```

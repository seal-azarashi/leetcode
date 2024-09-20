# 102. Binary Tree Level Order Traversal

LeetCode URL: https://leetcode.com/problems/binary-tree-level-order-traversal/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 10分ぐらい
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成する
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> nodesInEachLevel = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        while (!nodes.isEmpty()) {
            int nodesSizeSnapshot = nodes.size();
            List<Integer> valsInCurrentLevel = new LinkedList<>();
            for (int i = 0; i < nodesSizeSnapshot; i++) {
                TreeNode node = nodes.poll();
                valsInCurrentLevel.add(node.val);
                if (node.left != null) {
                    nodes.offer(node.left);
                }
                if (node.right != null) {
                    nodes.offer(node.right);
                }
            }
            nodesInEachLevel.add(valsInCurrentLevel);
        }
        return nodesInEachLevel;
    }
}
```

次のようなことを考えながら実装していました:

- 各レベルごとに走査を行うので BFS アプローチが適している
- "return the level order traversal of its nodes' values." と問題文にあるが、なんか英語がおかしい...?
    - traversal は走査することを表すので、これが返り値になるコレクション要素を表していたというのを example を見るまでわからなかった
    - 返り値を格納する変数名は levelOrderTraversal にしない方が良さそう (書いてる自分がよくわからなくなるので)
- node が empty になると constraints にあるのでこれに対処しなければ

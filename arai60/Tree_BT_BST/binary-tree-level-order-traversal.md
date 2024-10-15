# 102. Binary Tree Level Order Traversal

LeetCode URL: https://leetcode.com/problems/binary-tree-level-order-traversal/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 10分ぐらい
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
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
- ひとつのキューを使って実装したい
    - 再帰処理じゃないのでスタックオーバーフローのリスクがないため
    - インスタンスを new する処理は比較的重いので、レベルごとにこれを実施するのは避ける
- "return the level order traversal of its nodes' values." と問題文にあるが、なんか英語がおかしい...?
    - traversal は走査することを表すので、これが返り値になるコレクション要素を表していたというのを example を見るまでわからなかった
    - 返り値を格納する変数名は levelOrderTraversal にしない方が良さそう (書いてる自分がよくわからなくなるので)
- node が empty になると constraints にあるのでこれに対処しなければ
- リストの実装型は LinkedList が良さそう
    - 後ろに要素を追加していくだけなので、インデックスを使うためのオーバーヘッドが多い ArrayList は除外

## Step 2

### キューを用いた BFS アプローチ

Step 1 の解法から、一部変数名だけ修正しています。

- nodesInEachLevel -> valsInEachLevel
- nodesSizeSnapshot -> currentLevelNodeCount

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> valsInEachLevel = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        while (!nodes.isEmpty()) {
            int currentLevelNodeCount = nodes.size();
            List<Integer> valsInCurrentLevel = new LinkedList<>();
            for (int i = 0; i < currentLevelNodeCount; i++) {
                TreeNode node = nodes.poll();
                valsInCurrentLevel.add(node.val);
                if (node.left != null) {
                    nodes.offer(node.left);
                }
                if (node.right != null) {
                    nodes.offer(node.right);
                }
            }
            valsInEachLevel.add(valsInCurrentLevel);
        }
        return valsInEachLevel;
    }
}
```

### スタックを用いた DFS アプローチ

```java
class Solution {
    private record NodeLevel(TreeNode node, int level) {};

    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new ArrayList();
        }

        List<List<Integer>> valsInEachLevel = new ArrayList();
        Deque<NodeLevel> nodeLevelStack = new ArrayDeque();
        nodeLevelStack.push(new NodeLevel(root, 0));
        while (!nodeLevelStack.isEmpty()) {
            NodeLevel nodeLevel = nodeLevelStack.pop();
            if (valsInEachLevel.size() <= nodeLevel.level) {
                valsInEachLevel.add(new ArrayList());
            }
            valsInEachLevel.get(nodeLevel.level).add(nodeLevel.node.val);

            int nextLevel = nodeLevel.level + 1;
            if (nodeLevel.node.right != null) {
                nodeLevelStack.push(new NodeLevel(nodeLevel.node.right, nextLevel));
            }
            if (nodeLevel.node.left != null) {
                nodeLevelStack.push(new NodeLevel(nodeLevel.node.left, nextLevel));
            }
        }
        return valsInEachLevel;
    }
}
```

### 再帰関数を用いた DFS アプローチ

スタックオーバーフローのリスクはありますが、練習で書いてみました。

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、最大でノードの数だけスタックが生成される
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> valsInEachLevel = new ArrayList<>();
        findValuesInEachLevel(root, 0, valsInEachLevel);
        return valsInEachLevel;
    }

    private void findValuesInEachLevel(TreeNode node, int level, List<List<Integer>> valsInEachLevel) {
        if (valsInEachLevel.size() == level) {
            valsInEachLevel.add(new LinkedList<>());
        }

        valsInEachLevel.get(level).add(node.val);
        int nextLevel = level + 1;
        if (node.left != null) {
            findValuesInEachLevel(node.left, nextLevel, valsInEachLevel);
        }
        if (node.right != null) {
            findValuesInEachLevel(node.right, nextLevel, valsInEachLevel);
        }
    }
}
```

## Step 3

Step 1 と同じです。

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> valsInEachLevel = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        while (!nodes.isEmpty()) {
            int currentLevelNodeCount = nodes.size();
            List<Integer> valsInCurrentLevel = new LinkedList<>();
            for (int i = 0; i < currentLevelNodeCount; i++) {
                TreeNode node = nodes.poll();
                valsInCurrentLevel.add(node.val);
                if (node.left != null) {
                    nodes.offer(node.left);
                }
                if (node.right != null) {
                    nodes.offer(node.right);
                }
            }
            valsInEachLevel.add(valsInCurrentLevel);
        }
        return valsInEachLevel;
    }
}
```

## Step 4

### キューを用いた BFS アプローチ

次のレビューに対応しました:
- https://github.com/seal-azarashi/leetcode/pull/25#discussion_r1768932526

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> valsInEachLevel = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        while (!nodes.isEmpty()) {
            Queue<TreeNode> newNodes = new ArrayDeque<>();
            List<Integer> valsInCurrentLevel = new LinkedList<>();
            while (!nodes.isEmpty()) {
                TreeNode node = nodes.poll();
                valsInCurrentLevel.add(node.val);
                if (node.left != null) {
                    newNodes.offer(node.left);
                }
                if (node.right != null) {
                    newNodes.offer(node.right);
                }
            }
            valsInEachLevel.add(valsInCurrentLevel);
            nodes = newNodes;
        }
        return valsInEachLevel;
    }
}
```

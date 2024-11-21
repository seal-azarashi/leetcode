# 103. Binary Tree Zigzag Level Order Traversal

LeetCode URL: https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 5分ぐらい
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> zigzagLevelOrderVals = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        boolean isReversed = false;
        while (!nodes.isEmpty()) {
            int currentLevelNodeCount = nodes.size();
            List<Integer> valsInCurrentLevel = new LinkedList<>();
            for (int i = 0; i < currentLevelNodeCount; i++) {
                TreeNode node = nodes.poll();
                if (isReversed) {
                    valsInCurrentLevel.addFirst(node.val);
                } else {
                    valsInCurrentLevel.addLast(node.val);
                }
                if (node.left != null) {
                    nodes.offer(node.left);
                }
                if (node.right != null) {
                    nodes.offer(node.right);
                }
            }
            isReversed = !isReversed;
            zigzagLevelOrderVals.add(valsInCurrentLevel);
        }
        return zigzagLevelOrderVals;
    }
}
```

次のようなことを考えながら実装していました:

- 各レベルごとに走査を行うので BFS アプローチが適している
- ひとつのキューを使って実装したい
    - 再帰処理じゃないのでスタックオーバーフローのリスクがないため
    - インスタンスを new する処理は比較的重いので、レベルごとにこれを実施するのは避ける
- 並びの向きを制御するには 1 bit の情報量の値があればいいので boolean 値を使う
- 最後に reversed() メソッドで反転する方法も思いついたが、 boolean 値 isReversed のチェックを都度する方がオーバーヘッドが少ないはず
- リストの実装型は LinkedList が良さそう
    - Head もしくは tail に要素を追加していくだけなので、インデックスを使うためのオーバーヘッドが多い ArrayList は除外

## Step 2

### 再帰関数を用いた DFS アプローチ

スタックオーバーフローのリスクはありますが、練習で書いてみました。

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、最大でノードの数だけスタックが生成される
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> zigzagLevelOrderVals = new ArrayList<>();
        findValuesInEachLevel(root, 0, zigzagLevelOrderVals);
        return zigzagLevelOrderVals;
    }

    private void findValuesInEachLevel(TreeNode node, int level, List<List<Integer>> zigzagLevelOrderVals) {
        if (zigzagLevelOrderVals.size() == level) {
            zigzagLevelOrderVals.add(new LinkedList<>());
        }

        if (level % 2 == 0) {
            zigzagLevelOrderVals.get(level).addLast(node.val);
        } else {
            zigzagLevelOrderVals.get(level).addFirst(node.val);
        }
        
        int nextLevel = level + 1;
        if (node.left != null) {
            findValuesInEachLevel(node.left, nextLevel, zigzagLevelOrderVals);
        }
        if (node.right != null) {
            findValuesInEachLevel(node.right, nextLevel, zigzagLevelOrderVals);
        }
    }
}
```

### Step 1 を読みやすくする

オーバーヘッドを少しでも減らそうと工夫していましたが、それで削減できるのは微々たるものなので、読みやすさをとってレベルごとの処理の最後に reversed() メソッドを呼ぶように変えました。  
参考: https://github.com/fhiyo/leetcode/pull/29#discussion_r1660231840

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> zigzagLevelOrderVals = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        boolean isReversed = false;
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

            if (isReversed) {
                valsInCurrentLevel = valsInCurrentLevel.reversed();
            }
            isReversed = !isReversed;

            zigzagLevelOrderVals.add(valsInCurrentLevel);
        }
        return zigzagLevelOrderVals;
    }
}
```

## Step 3

```java
// 時間計算量: O(n): 全部のノードを一度走査する
// 空間計算量: O(n): 全部のノードが入ったリストを作成するのに加え、キューが必要
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }

        List<List<Integer>> zigzagLevelOrderVals = new LinkedList<>();
        Queue<TreeNode> nodes = new ArrayDeque<>();
        nodes.offer(root);
        boolean isReversed = false;
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

            if (isReversed) {
                valsInCurrentLevel = valsInCurrentLevel.reversed();
            }
            isReversed = !isReversed;

            zigzagLevelOrderVals.add(valsInCurrentLevel);
        }
        return zigzagLevelOrderVals;
    }
}
```

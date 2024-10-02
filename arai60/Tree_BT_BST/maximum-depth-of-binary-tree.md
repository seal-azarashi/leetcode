# 104. Maximum Depth of Binary Tree

LeetCode URL: https://leetcode.com/problems/maximum-depth-of-binary-tree/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 解いた時間: 10分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Deque<TreeNode> treeNodes = new ArrayDeque<>();
        treeNodes.addLast(root);
        int maxDepth = 0;
        while (!treeNodes.isEmpty()) {
            int levelNodeCountSnapshot = treeNodes.size();
            for (int i = 0; i < levelNodeCountSnapshot; i++) {
                TreeNode node = treeNodes.removeFirst();
                if (node.left != null) {
                    treeNodes.addLast(node.left);
                }
                if (node.right != null) {
                    treeNodes.addLast(node.right);
                }
            }
            maxDepth++;
        }
        return maxDepth;
    }
}
```

次のようなことを考えながら実装していました:

- Queue を用いた BFS と再帰関数による DFS の2つのアプローチを思いつく  
- 前者はスタックオーバーフローのリスクが無く、また実装も後者と比べ理解しやすい内容になると考え、そちらで書いてみた
    - Leetcode の constraints 上はスタックオーバーフローのリスクはほぼ無いと考えらるが、 [oda さんのコメント](https://github.com/kazukiii/leetcode/pull/22/files#r1667746480)にある通り「この環境では大丈夫」なコードはいつ自分の足を撃ち抜いて来るかわからないので避けた
- 次の理由でキュー (ArrayDeque) へは null を挿入していない:
    - そもそも ArrayDeque の各メソッドは null を挿入することを許容しない
    - null チェックをイテレーションごとに都度実施しなくていいなら、そうする越したことはないと思う
    - [ArrayDeque はドキュメント冒頭にある通り早い](https://docs.oracle.com/en%2Fjava%2Fjavase%2F22%2Fdocs%2Fapi%2F%2F/java.base/java/util/ArrayDeque.html)ので、その利点を捨ててまで、例えば null 許容する LinkedList を implementing class にすることもないかなと考える

## Step 2

### 再帰関数による DFS

```java
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        return traverseDepthRecursively(root);
    }

    private int traverseDepthRecursively(TreeNode node) {
        int leftDepth = 0;
        if (node.left != null) {
            leftDepth = traverseDepthRecursively(node.left);
        }
        int rightDepth = 0;
        if (node.right != null) {
            rightDepth = traverseDepthRecursively(node.right);
        }
        return Math.max(leftDepth, rightDepth) + 1;
    }
}
```

次のようなことを考えながら書いていました:

- [ryotaro さんの実装](https://github.com/Ryotaro25/leetcode_first60/blob/94e08f19a2e6d4bd2f7eb999a14d48732ebfa32e/104.MaximumDepthofBinaryTree/step4.cpp)のように maxDepth() を再帰関数にすることも検討したが次の理由でそのようにはしなかった:
    - 引数名 root が値を正しく表すのは最初だけ
    - 実装によっては root に対する毎回の null チェックは不要になる
- 三項演算子にすれば traverseDepthRecursively() の行数を減らすことも出来るが、改行のしかたなど、書き方が人によって結構分かれそうで、開発者の数が多い場合はあまり好ましくないと感じた
    - 1行あたりの文字数が多くなるので、 linter によって見づらい形に改行されたりもしそう
    - 今の書き方ならそんなに人による差異は出ないはず

### 深さを付与した node とスタックを用いた DFS

```java
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    private record TreeNodeDepth(TreeNode node, int depth) {};

    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Deque<TreeNodeDepth> treeNodeDepthStack = new ArrayDeque<>();
        treeNodeDepthStack.push(new TreeNodeDepth(root, 1));
        int maxDepth = 0;
        while (!treeNodeDepthStack.isEmpty()) {
            TreeNodeDepth treeNodeDepth = treeNodeDepthStack.pop();
            maxDepth = Math.max(maxDepth, treeNodeDepth.depth);
            if (treeNodeDepth.node.left != null) {
                treeNodeDepthStack.push(new TreeNodeDepth(treeNodeDepth.node.left, treeNodeDepth.depth + 1));
            }
            if (treeNodeDepth.node.right != null) {
                treeNodeDepthStack.push(new TreeNodeDepth(treeNodeDepth.node.right, treeNodeDepth.depth + 1));
            }
        }
        return maxDepth;
    }
}
```

- そういえば最初に解いたときにスタックで DFS をする方法は思いつかなかった
- 上記の方法は多くの方が実装されてたので、自分もやってみた

## Step 3

```java
// 解いた時間: 4分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        Queue<TreeNode> treeNodes = new ArrayDeque<>();
        treeNodes.offer(root);
        int maxDepth = 0;
        while (!treeNodes.isEmpty()) {
            int currentLevelNodesCount = treeNodes.size();
            for (int i = 0; i < currentLevelNodesCount; i++) {
                TreeNode node = treeNodes.poll();
                if (node.left != null) {
                    treeNodes.offer(node.left);
                }
                if (node.right != null) {
                    treeNodes.offer(node.right);
                }
            }
            maxDepth++;
        }
        return maxDepth;
    }
}
```

- 読み手にとって処理のイメージが掴みやすくなるように treeNodes のインターフェースを Queue に変える
- levelNodeCountSnapshot を currentLevelNodesCount に修正

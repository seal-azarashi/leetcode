# 108. Convert Sorted Array to Binary Search Tree

LeetCode URL: https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

```java
// 時間計算量: O(n): 配列の各要素に対して一回ずつ操作を実施
// 空間計算量: O(n): 配列の要素と同じ数のノードを持つツリーを生成
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        return buildBSTRecursively(nums, 0, nums.length - 1);
    }

    private TreeNode buildBSTRecursively(int[] nums, int start, int end) {
        if (start > end) {
            return null;
        }

        int middle = start + (end - start) / 2;
        TreeNode node = new TreeNode(nums[middle]);
        node.left = buildBSTRecursively(nums, start, middle - 1);
        node.right = buildBSTRecursively(nums, middle + 1, end);
        return node;
    }
}
```

次のようなことを考えながら実装していました:

- 配列の要素が strictly increasing order かつ nullable でないのでシンプルな実装にできそう
- 木の深さは最大でも14なので、再帰的に実装してもスタックオーバーフローにはならないはず
    - そもそも Java の配列は要素数が最大で `2^31 - 1` であるため、最大でも木の深さは31にしかならない
    - 基本的に10000ぐらいはスタックを積んでも大丈夫とされているので、スタックオーバーフローを考慮にいれる必要はないかな
- middle の算出について、単純に `(start + end) / 2` とすると要素数が多い場合に整数値の最大数 (`2^31 - 1`) を超えて integer overflow (桁溢れ) になってしまうので少し工夫が必要だ
    - ちなみに範囲を超えた小さい値をアサインしようとした際に起こるのも同じく integer overflow: https://en.wikipedia.org/wiki/Integer_overflow
    - Underflow (下位桁溢れ) という言葉もあるが、これは型が取り扱い可能な絶対値の最小値を下回り、正しく計算できなくなることを指すので気をつけたい: https://teach-ict.com/2016/A_Level_Computing/OCR_H446/1_4_data_types_structures_algorithms/141_data_types/norm/miniweb/pg2.php
- Constraints には配列が null になるもしくは要素数が 0 になることはないとあるけど、実際のプロダクトで運用することを想定すると、引数に想定外のものが渡されても可能な限り対応できるようにしておくのが望ましいので、これらもチェックしたい

## Step 2

### 閉区間インデックスを用いた二分探索的アプローチ

Step 1 の解法をアップデートします。

```java
// 時間計算量: O(n): 配列の各要素に対して一回ずつ操作を実施
// 空間計算量: O(n): 配列の要素と同じ数のノードを持つツリーを生成
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        return buildBSTRecursively(nums, 0, nums.length - 1);
    }

    private TreeNode buildBSTRecursively(int[] nums, int left, int right) {
        if (right < left) {
            return null;
        }

        int middle = left + (right - left) / 2;
        TreeNode node = new TreeNode(nums[middle]);
        node.left = buildBSTRecursively(nums, left, middle - 1);
        node.right = buildBSTRecursively(nums, middle + 1, right);
        return node;
    }
}
```

細かいですが以下修正しています:

- 区間を表す各変数を left, right に改名
- 有効な区間かどうかの判定を `right < left` と記述
    - 数直線上のイメージから小なりないし小なりイコール記号が好まれるのを今までのレビューから感じたので、それに合わせた
    - `right` が左側に来る違和感から、何をやっているのかのイメージがつきやすくなるように思えた

### 右半開区間インデックスを用いた二分探索的アプローチ

よくある配列操作と同じインデックスの指定の仕方ですが、この問題に使うのは閉区間の方のアプローチと比べるとやや直感的でない印象でした。

```java
// 時間計算量: O(n): 配列の各要素に対して一回ずつ操作を実施
// 空間計算量: O(n): 配列の要素と同じ数のノードを持つツリーを生成
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        return buildBSTRecursively(nums, 0, nums.length);
    }

    private TreeNode buildBSTRecursively(int[] nums, int left, int right) {
        if (right <= left) {
            return null;
        }

        int middle = left + (right - left) / 2;
        TreeNode node = new TreeNode(nums[middle]);
        node.left = buildBSTRecursively(nums, left, middle);
        node.right = buildBSTRecursively(nums, middle + 1, right);
        return node;
    }
}
```

### 再帰的に配列の要素をコピーしていくアプローチ

ヘルパー関数を使わなくても書けるなと思い書いてみましたが、インデックスを渡せないためコピーが発生してしまうので、計算量に結構差が出ました。

```java
// 時間計算量: O(n log n): 各要素に対する操作と log n の深さのツリーの各レベルで行われる配列コピー操作
// 空間計算量: O(n log n): log n の深さのツリーの各レベルで元配列のほぼ全要素をコピー
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        int middle = nums.length / 2;
        TreeNode node = new TreeNode(nums[middle]);
        node.left = sortedArrayToBST(Arrays.copyOfRange(nums, 0, middle));
        node.right = sortedArrayToBST(Arrays.copyOfRange(nums, middle + 1, nums.length));
        return node;
    }
}
```

### スタックを用いたアプローチ

```java
// 時間計算量: O(n): 配列の各要素に対して一回ずつ操作を実施
// 空間計算量: O(n): 配列の要素と同じ数のノードを持つツリーを生成 + スタックの容量
class Solution {
    private static final int DUMMY_VAL = 0;
    private record NodeWithRange(TreeNode node, int left, int right) {}

    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }

        TreeNode root = new TreeNode(DUMMY_VAL);
        ArrayDeque<NodeWithRange> stack = new ArrayDeque<>();
        stack.push(new NodeWithRange(root, 0, nums.length - 1));

        while (!stack.isEmpty()) {
            NodeWithRange nodeWithRange = stack.pop();
            TreeNode node = nodeWithRange.node();
            int left = nodeWithRange.left();
            int right = nodeWithRange.right();

            int middle = left + (right - left) / 2;
            node.val = nums[middle];
            if (left <= middle - 1) {
                node.left = new TreeNode(DUMMY_VAL);
                stack.push(new NodeWithRange(node.left, left, middle - 1));
            }
            if (middle + 1 <= right) {
                node.right = new TreeNode(DUMMY_VAL);
                stack.push(new NodeWithRange(node.right, middle + 1, right));
            }
        }

        return root;
    }
}
```

## Step 3

最も直感的に思えて、かつ計算量に優れる閉区間インデックスを用いた二分探索的アプローチを選びました。

```java
// 解いた時間: 約4分
// 時間計算量: O(n): 配列の各要素に対して一回ずつ操作を実施
// 空間計算量: O(n): 配列の要素と同じ数のノードを持つツリーを生成
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums == null || nums.length == 0) {
            return null;
        }
        
        return sortedArrayToBSTRecursively(nums, 0, nums.length - 1);
    }

    private TreeNode sortedArrayToBSTRecursively(int[] nums, int left, int right) {
        if (right < left) {
            return null;
        }

        int middle = left + (right - left) / 2;
        TreeNode node = new TreeNode(nums[middle]);
        node.left = sortedArrayToBSTRecursively(nums, left, middle - 1);
        node.right = sortedArrayToBSTRecursively(nums, middle + 1, right);
        return node;
    }
}
```

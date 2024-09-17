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
- 木の深さは最大でも14なので、再帰的に実装すればスタックオーバーフローにはならないはず
    - そもそも Java の配列は要素数が最大で `2^31 - 1` で、最大でも木の深さは31にしかならない
    - 基本的に10000ぐらいはスタックを積んでも大丈夫なので、スタックオーバーフローを考慮にいれる必要はないかな
- middle の算出について、単純に `(start + end) / 2` とすると要素数が多い場合に整数値の最大数 (`2^31 - 1`) を超えて integer overflow (桁溢れ) になってしまうので少し工夫が必要だ
    - ちなみに範囲を超えた小さい値をアサインしようとした際に起こるのも同じく integer overflow: https://en.wikipedia.org/wiki/Integer_overflow
    - Underflow (下位桁溢れ) という言葉もあるが、これは型が取り扱い可能な絶対値の最小値を下回り、正しく計算できなくなることを指すので気をつけたい: https://teach-ict.com/2016/A_Level_Computing/OCR_H446/1_4_data_types_structures_algorithms/141_data_types/norm/miniweb/pg2.php
- Constraints には配列が null になるもしくは要素数が 0 になることはないとあるけど、実際のプロダクトで運用することを想定すると、引数に想定外のものが渡されても可能な限り対応できるようにしておくのが望ましいので、これらもチェックしたい

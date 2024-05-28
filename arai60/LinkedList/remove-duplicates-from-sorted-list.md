# 83. Remove Duplicates from Sorted List

LeetCode URL: https://leetcode.com/problems/remove-duplicates-from-sorted-list/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

尚、各設問で扱われる ListNode のクラス定義は、 LeetCode のページで言語を Java にしたら出てくる次のコメントの通りです。

```
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
```

## Recursive

再帰関数を用いた実装です。

Time Complexity: O(n), Space Complexity: O(1)

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null || head.next == null) return head;

        head.next = deleteDuplicates(head.next);

        return head.val == head.next.val ? head.next : head;
    }
}
```

## Iterative

反復的なアプローチでの実装です。

Time Complexity: O(n), Space Complexity: O(1)

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (node != null && node.next != null) {
            if (node.val == node.next.val) {
                node.next = node.next.next;
                continue;
            }

            node = node.next;
        }

        return head;
    }
}
```

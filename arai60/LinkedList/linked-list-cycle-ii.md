# 142. Linked List Cycle II

LeetCode URL: https://leetcode.com/problems/linked-list-cycle-ii/description/

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

## Floyd's tortoise and hare algorithm

[Floyd's tortoise and hare algorithm](https://www.geeksforgeeks.org/floyds-cycle-finding-algorithm/) を用いた解法です。

Time Complexity: O(n), Space Complexity: O(1)

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) break;
        }

        if (fast == null || fast.next == null) return null;

        while (head != slow) {
            head = head.next;
            slow = slow.next;
        }

        return head;
    }
}
```

## HashSet

HashSet に Node を格納し、走査中のものと同じ値が格納されていないか (サイクルが発生していないか) をチェックします。  
最初に見つかった値がサイクルの始まりになるので、 (サイクルがある場合は) それを返します。

Time Complexity: O(n), Space Complexity: O(n)

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();

        while (head != null) {
            if (set.contains(head)) return head;

            set.add(head);

            head = head.next;
        }

        return head;
    }
}
```

# 141. Linked List Cycle

LeetCode URL: https://leetcode.com/problems/linked-list-cycle/description/

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

### Recursive

再帰関数を使った実装になります。 

``` java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null) return false;

        return recursive(head, head.next);
    }

    private boolean recursive(ListNode slow, ListNode fast) {
        if (fast == null || fast.next == null) return false;

        if (slow == fast) return true;

        return recursive(slow.next, fast.next.next);
    }
}
```

### Iterative

反復的なアプローチで実装しています。 

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) return true;
        }

        return false;
    }
}
```

## HashSet

HashSet に Node を格納し、走査中のものと同じ値が格納されていないか (サイクルが発生していないか) をチェックします。

Time Complexity: O(n), Space Complexity: O(n)

## HashSet: Recursive

再帰関数を使った実装になります。 

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();

        return recursive(set, head);
    }

    private boolean recursive(Set<ListNode> set, ListNode node) {
        if (node == null) return false;

        if (set.contains(node)) return true;

        set.add(node);

        return recursive(set, node.next);
    }
}
```

## Iterative

反復的なアプローチで実装しています。

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();
        
        ListNode node = head;
        while (node != null) {
            if (set.contains(node)) return true;

            set.add(node);

            node = node.next;
        }

        return false;
    }
}
```

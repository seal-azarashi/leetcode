この問題は Java で解いています。  
ListNode のクラス定義は、 LeetCode のページで言語を Java にしたら出てくる次のコメントの通りです。

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

- Use Floyd's tortoise and hare algorithm: https://www.geeksforgeeks.org/floyds-cycle-finding-algorithm/
- Time Complexity: O(n), Space Complexity: O(1)

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

## Iterative

- Use Floyd's tortoise and hare algorithm: https://www.geeksforgeeks.org/floyds-cycle-finding-algorithm/
- Time Complexity: O(n), Space Complexity: O(1)

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

- Store traversed Node in HashSet
- Time Complexity: O(n), Space Complexity: O(n)

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

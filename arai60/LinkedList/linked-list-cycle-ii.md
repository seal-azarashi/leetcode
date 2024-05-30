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

## Step 1

かかった時間: 7 m 37 s

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();

        return recursive(set, head);
    }

    private ListNode recursive(Set<ListNode> set, ListNode node) {
        if (node == null) return null;

        if (set.contains(node)) return node;

        set.add(node);

        return recursive(set, node.next);
    }
}
```

※[以前空行が多いこと等々指摘されました](https://discord.com/channels/1084280443945353267/1245404801177616394/1245412412123779113)が、これは指摘を受ける前に書いたものをそのまま乗せてますのでその指摘が反映されてません

思考ログ:

- HashSet を使った解法しか思いつかなかったのでそれで解くことを決める
    - ノードを走査していって、初めて HashSet に格納済のノードに再訪したら、それをサイクルの始まりとみなす
- なんとなく再帰の書き方に慣れてて、どうせ後でループで書き直すからとひとまず再帰で書き始める (この点は[以前のレビュー](https://discord.com/channels/1084280443945353267/1245404801177616394/1245560536335253504)で指摘された通り反省ポイント)
- 特に詰まらずに最初の submission でパスした

## Step 2

- 他の解法を見ていて、 [Floyd's tortoise and hare algorithm](https://www.geeksforgeeks.org/floyds-cycle-finding-algorithm/) を用いた解法があることを知る
    1. slow, fast pointer があるノードで出会うことをもってサイクルの検知をする
    2. 先頭ノードにポインタを置き、上記のノードにあるポインタと共に1ずつ進めていく
    3. 2 で動く両ポインタが出会ったノードがサイクルの始まり
- 一度再帰で書こうとするも見通しのいいコードが思いつかず、ループで次のように書いたら一度でパスした

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) return null;

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

- detectCycle() 一行目の null チェックが不要だと気づいて消す

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

- 再帰的な書き方で見通しのいい実装例がないか探すも特に見つけられない
- 探している最中に、そもそも再帰で書くとスタック領域を使う分パフォーマンスが劣化すると気づく
- 上記勘案して、ループと再帰両方で書けるようになる必要があるとは理解しつつも、この問題においては解法として再帰を選択することはないだろうと判断
- Step 3 ではベストなソリューションである上記のループによる解法を扱うことに決める
- そういえば HashSet を用いてループで処理する解法を用意してなかったので、以下の通り書いて一回目の submission でパスさせた

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

※ここまで約15分

## Step 3

上記の Floyd's ~ を用いたループによる解法を、ミスなしで3連続アクセプトさせた。  
1回目と2回目は2分ほどでパスし、3回目はミスりたくないので慎重にやったら5分かかっていた。

## Step 4

- こチラの問題を解いた後に別のプルリクエストで受けたレビューを元に、上に挙げた3つの解法をそれぞれ次のように書き直した

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();
        return recursive(set, head);
    }

    private ListNode recursive(Set<ListNode> set, ListNode node) {
        if (node == null) return null;
        if (set.contains(node)) return node;
        set.add(node);
        return recursive(set, node.next);
    }
}
```

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

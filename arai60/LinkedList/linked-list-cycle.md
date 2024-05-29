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

## Step 1

かかった時間: 10 m 28 s

```java
// Time Complexity: O(n), Space Complexity: O(1)
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;

        return recursive(head, head.next);
    }

    private boolean recursive(ListNode one, ListNode two) {
        if (one == null || two == null || two.next == null) return false;
        
        if (one == two) return true;

        return recursive(one.next, two.next.next);
    }
}
```

思考ログ:

- 解いたことのある問題だったのに気づく
- Floyd's tortoise and hare algorithm を使った解法でいけるはずと思い、答えは見ずに書ききろうと決める
    - この解法なら Time Complexity: O(n), Space Complexity: O(1)
    - パスするはず
- recursive な書き方が思いついたので書き始める
- 最初の問題なので景気づけに絶対解答してやろうと意気込む
- 5分ぐらいでパスするであろうコードが出来る
- ミスりたくないので何度も確認したりリファクタリング案を考えてたら10分経過
- submit したらパスした

## Step 2

既存のコードを整える (約5分)

- recursive な書き方でもっときれいに出来ないか試行錯誤
- 結局元の解答のままにすることに決める

他の解法の調査 (約2分)

- HashMap を使った解法があるのは把握していたのでそれは書こうと思う
- 他に何か知らないものがないか確認するため LeetCode の Solutions 欄を見る
- 上記解答を iterative に書いてる例をみつける
- 他に有力なものはなさそうだったので終わり

別解法を書く:

```java
// Floyd's tortoise and hare algorithm を iterative に書く
// Time Complexity: O(n), Space Complexity: O(1)
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

- 自分で考えず、 Solution 欄にあったものをまるごとコピー
- コピーしたものを自分好みに書き換えた
- かかったのは数分程度

```java
// HashSet を使う
// Time Complexity: O(n), Space Complexity: O(n)
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

- 簡単なのは目に見えてたのでこちらは自分で解いてみる
- 計測してなかったが3分程度だったと記憶している
- もう数分かけて Solution 欄の他の解答を一応見るも、あまり参考にせずほぼ自分の案そのままで完成とした

## Step 3 

**上記全ての解法それぞれを**、連続で3回、10分以内に、ひとつもエラーを出さずにアクセプトされるまで繰り返し解いた。それぞれ数回エラーを出し、ひとつあたり10分以上かかった。

## Step 4

[oda さんの最初のレビュー](https://discord.com/channels/1084280443945353267/1245404801177616394/1245412412123779113)を踏まえ

- ログを追加
- 空行が多いとのしてきがあったので、それぞれ以下のように修正した

```java
// Time Complexity: O(n), Space Complexity: O(1)
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;
        return recursive(head, head.next);
    }

    private boolean recursive(ListNode one, ListNode two) {
        if (one == null || two == null || two.next == null) return false;
        if (one == two) return true;
        return recursive(one.next, two.next.next);
    }
}
```

```java
// Floyd's tortoise and hare algorithm を iterative に書く
// Time Complexity: O(n), Space Complexity: O(1)
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

```java
// HashSet を使う
// Time Complexity: O(n), Space Complexity: O(n)
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

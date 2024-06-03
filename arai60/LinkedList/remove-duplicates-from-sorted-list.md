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

## Step 1

かかった時間: 4 m 58 s

```java
// Time Complexity: O(n), Space Complexity: O(n)
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (true) {
            if (node == null || node.next == null) break;

            while (node.next != null && node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }

        return head;
    }
}
```

思考ログ:

- まず、次の解き方を思いついたのでループで実装を試みる:
    - 各ノードの val が、その次のノードの val と等しければ、さらにその次のノードに連結する (node.next に node.next.next を代入する)
    - もし node, node.next が null の場合、走査を終えて head を返す
- 実装は次のようになった

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (true) {
            if (node == null || node.next == null) break;

            if (node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }

        return head;
    }
}
```

- ひとつテストケースが失敗したので原因を考える:
    - head     = [1,1,1]
    - Output   = [1,1]
    - Expected = [1]
- 最後のノードが連続する val の重複した node の一部だと、末尾の要素を排除できない欠点があることに気づく
- 実装をどう修正するか考える上で既存の処理を全体的に見直したところ、一回の while 文の iteration で、ノードの走査を1ずつ進めていることに違和感を覚える
- テストケース失敗の原因となった上記の欠点がなく、かつ一回の iteration で重複する要素を全て除くよう、次のように修正した

```java
// Time Complexity: O(n), Space Complexity: O(n)
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (true) {
            if (node == null || node.next == null) break;

            while (node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }

        return head;
    }
}
```

- 既存のテストケースを実行したら null pointer exception が発生
- 2つめの while 文の実行時に null チェックをしていないことが原因だったので修正
- このステップの最上部に書いた実装の通りに修正し、パスすることを確認した

## Step 2

### Step 1 で書いた実装の修正

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (node != null && node.next != null) {
            while (node.val == node.next.val) {
                node.next = node.next.next;
                if (node.next == null) return head;
            }
            node = node.next;
        }

        return head;
    }
}
```

思考ログ:

- 2つめの while 文に `node.next != null` があるが、これは null ~ を防ぐためにつけたものだった
- やりたかったのは「ここで node.next が null になったら (もう走査対象がないため) 処理を中断する」ことなので、これに沿うような形に変えてみる

```java
// Time Complexity: O(n), Space Complexity: O(n)
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        // (略)

            while (node.val == node.next.val) {
                node.next = node.next.next;
                if (node.next == null) return head;
            }

        // (略)
    }
}
```

- 見通しが良くなった気がする
- 1つめの while 文の終了条件の書き方も変えたくなったので、このステップ冒頭に示したコードのように修正した
- 処理を口頭で説明するならば、次のようにするのが自然かなと考えるので、これで素直な処理になったように思う
    - 先頭のノードから順に、その次のノードとペアで処理していきます。末尾にたどりついたら head を返します。
    - 処理中、対象のノードの val が次のノードのものと等しかったら、さらにその次のノードを next に代入します。これを val と next が異なるものになるまで繰り返します。next が null になったらもう操作対象がないので、 head を return しましょう。


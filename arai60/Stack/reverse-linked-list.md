# 206. Reverse Linked List

LeetCode URL: https://leetcode.com/problems/reverse-linked-list/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

尚、各設問で扱われる ListNode のクラス定義は、 LeetCode のページで言語を Java にしたら出てくる次のコメントの通りです。

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
```

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 約14分
// 時間計算量: O(n), 空間計算量: O(1)
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) return head;

        ListNode previous = null;
        while (true) {
            ListNode node = head;
            ListNode nextNode = node.next;
  
            node.next = previous;
            if (nextNode == null) return head;

            previous = node;
            head = nextNode;
        }
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- まず、次の解き方を思いついたのでループで実装を試みる:
    - 3つのポインタ、 previous, node, nextNode を使う
        - node: あるイテレーションで着目しているノード
        - previous: 元のリストにおいて、 node の一つ前に位置するノード
        - nextNode: 元のリストにおいて、 node の一つ後ろに位置するノード
    - あるイテレーションにおける処理の手順は次の通り:
        - nextNode に node.next を代入する (次の処理で node.next の中身が変わるため)
        - node の向き先を previous に変える
        - previous, node を次に進める (previous には node を代入、 node には nextNode を代入)
- 今回 head のポインタを保持し続ける必要がないと気づき、上記 node の代わりに head を使おうと考える
    - 最終的に返されるリバース済のリストの先頭を指すので、 head という名前には違和感がないと判断
- 一回目のイテレーションで previous には null が代入されている必要があるため、 while 文開始より前に previous を宣言
- while 文の終了条件は、最後のイテレーションで head が previous を向いており、かつ nextNode が null であることだと判断
- while 文条件式で上記を表現するより、処理中に判定した方が見通しが良くなりそうだと判断し、条件式は (true) とする
- 一度 submit するも、要素数が1以下のときの考慮が出来ておらず fail
- メソッド1行目に上記考慮した処理を追加して submit したらパスした

## Step 2

2つ実装しました。

まずは Step 1 で書いたメソッドの修正版です。

```java
// 時間計算量: O(n), 空間計算量: O(1)
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode previous = null, node = head;
        while (node != null) {
            ListNode nextNode = node.next;
            node.next = previous;
            previous = node;
            node = nextNode;
        }

        return previous;
    }
}
```

再帰を用いた実装も用意しています。

```java
// 時間計算量: O(n), 空間計算量: O(n)
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverseListRecursively(null, head);
    }

    private ListNode reverseListRecursively(ListNode previous, ListNode node) {
        if (node == null) return previous;

        ListNode nextNode = node.next;
        node.next = previous;
        previous = node;
        node = nextNode;
        return reverseListRecursively(previous, node);
    }
}
```

### 思考ログ

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- まずは Step 1 で書いたコードを見直す
- node の代わりに head を使おうと途中で思い直したのにもかかわらず、変数 node が中途半端に残っているので削除

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) return head;

        ListNode previous = null;
        while (true) {
            ListNode nextNode = head.next;
            head.next = previous;

            if (nextNode == null) return head;

            previous = head;
            head = nextNode;
        }
    }
}
```

- 一行目にエッジケースの処理が入っているのと、 while の条件式が (true) となっているのを読みづらいと感じる
- while 終了時にリバースしたリスト先頭のノードのポインタを返したいが、それをするには previous を返せばよいと気づく
- そうなるともう head はリバースしたリストの先頭でもなんでもないので、感覚的に previous の後ろのノードは node とした方がいいと判断
- 上記踏まえてこのステップで最初に示した形に修正した
- 加えて、再帰での実装を行う

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverseListRecursively(null, null, head);
    }

    private ListNode reverseListRecursively(ListNode previous, ListNode node, ListNode head) {
        if (head == null) return node;

        ListNode nextNode = head.next;
        node = head;
        node.next = previous;
        previous = node;
        head = nextNode;
        return reverseListRecursively(previous, node, head);
    }
}
```

- ポイントを3つ宣言してしまったが2つでよいはずと気づく
- ポインタが3つあるのが煩雑に感じ、このステップで最初に示した形に修正した

## Step 3

ループを用いた処理: 1回目は4分、2, 3回目は1分半ほどで完了。
再帰を用いた処理: 約4分半、2,3回目は2分ほどで完了。

## Step 4

以下のレビューを参考に、ループを用いた実装を修正しました。

- https://github.com/seal-azarashi/leetcode/pull/7#discussion_r1638023615

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode previous = null, node = head;
        while (node != null) {
            ListNode next = node.next;
            node.next = previous;
            previous = node;
            node = next;
        }

        return previous;
    }
}
```

加えて、共有して頂いた[こちらのコメント](https://github.com/goto-untrapped/Arai60/pull/27/files/14646ec0859dd9411e6983bf6c63e6f15a1f9f32#r1638693522)における 2 の方の実装を追加しました。

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        // こちらの判定は初回に一度実施すれば良いため、reverseListRecursively 内部ではなくここで実施
        if (head == null) return null;
        
        return reverseListRecursively(head);
    }

    private ListNode reverseListRecursively(ListNode node) {
        if (node.next == null) return node;

        ListNode reversedListHead = reverseListRecursively(node.next);
        node.next.next = node;
        node.next = null;
        return reversedListHead;
    }
}
```

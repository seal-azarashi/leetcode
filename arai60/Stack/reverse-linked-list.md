# 206. Reverse Linked List

LeetCode URL: https://leetcode.com/problems/reverse-linked-list/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 約14分
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
- 一回目のイテレーションで previous には null が代入されている必要があるため、 while 文開始より前に previous を宣言
- while 文の終了条件は、最後のイテレーションで head が previous を向いており、かつ nextNode が null であることだと判断
- while 文条件式で上記を表現するより、処理中に判定した方が見通しが良くなりそうだと判断し、条件式は (true) とする
- 一度 submit するも、要素数が1以下のときの考慮が出来ておらず fail
- メソッド1行目に上記考慮した処理を追加して submit したらパスした

# 2. Add Two Numbers

LeetCode URL: https://leetcode.com/problems/add-two-numbers/

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
// 実装にかかった時間: 12 m 57 s
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode();

        ListNode node = dummyHead;
        boolean isCarrying = false;
        while (l1 != null || l2 != null) {
            int sum = 0;
            if (l1 == null) {
                sum = l2.val;
            } else if (l2 == null) {
                sum = l1.val;
            } else {
                sum = l1.val + l2.val;
            }

            int val = 0;
            sum = isCarrying ? sum + 1 : sum;
            if (sum > 9) {
                val = sum % 10;
                isCarrying = true;
            } else {
                val = sum;
                isCarrying = false;
            }

            node.next = new ListNode(val);
            node = node.next;
            l1 = l1 == null ? null : l1.next;
            l2 = l2 == null ? null : l2.next;
        }
        if (isCarrying) node.next = new ListNode(1);

        return dummyHead.next;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- いい解き方が思いつかなかったので、約30分かけて模範解答の内容を理解する
- 模範解答を理解したところで自分で解き始める
- 手順が多いので、実装中に重要な部分を忘れないよう大まかな処理の流れを次のようにコードコメントとして残す:

```
// add vals from the head

// Create dummyHead
// Create node substituting dummyHead
// at each iteration (while no one of l1 l2 is left):
// - add vals
// - if vals isCarrying, add one to the sum
// - if the sum is greater than 9, make the putting value `sum % 10` and make isCarrying true
// - if not, leave the putting value as it is and make isCarrying false
// - put new node with the value and attach it as node.next
// - move the pointer of node, l1, l2
// if isCarrying, add 1 as node.next
// return dummyHead.next
```

- 書き出して頭が整理されたのか、上記のコードコメントはほとんど見ずに解答を書いていく
- 上記コードコメントに反映されていなかったケースの考慮も適宜加えつつ実装
- 最初の submit でパスした

## Step 2

次のように Step 1 で書いた実装を修正しました。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummyHead = new ListNode();

        ListNode node = dummyHead;
        int carry = 0;
        while (l1 != null || l2 != null) {
            int sum = carry;
            if (l1 != null) {
                sum += l1.val;
                l1 = l1.next;
            }
            if (l2 != null) {
                sum += l2.val;
                l2 = l2.next;
            }

            node.next = new ListNode(sum % 10);
            node = node.next;
            carry = sum / 10;
        }
        if (carry != 0) node.next = new ListNode(1);

        return dummyHead.next;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- Step 1 で書いたコードはメソッド部分だけで約34行あり、これは長すぎると感じる
- 実装が必要以上に手続き的というか、まるで口頭で述べる処理の方法をそのまま一行一行書いていったようで、必要以上に行数を重ねてしまっているような印象を受ける
- (この時点でこれをしてしまったのは良くなかったと今は反省しているが) ChatGPT に改善案を聞いてみたところ、次の改善案が出る:
    1. 繰り上がりがあることを boolean 値で表すのでなく、繰り上がりの数を変数 carry に整数値として保持する
    2. l1, l2 の扱いが冗長なので、それぞれ null でない場合に sum に加算した後ポインタをひとつ進めるようにする
    3. node.next に代入する値を `sum % 10` にする (これにより、繰り上がりがある場合とない場合それぞれで入れるべき値が入るようになる) 
- 最終的な修正結果がこのステップで最初に示したコードとなる
- ChatGPT に聞く前にもう10分でも試行錯誤の時間を取れば良かったと反省

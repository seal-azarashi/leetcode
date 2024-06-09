# 82. Remove Duplicates from Sorted List II

LeetCode URL: https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/

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

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 21 m 32 s
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode node = dummy;
        while (node != null && node.next != null && node.next.next != null) {
            if (node.next.val == node.next.next.val) {
                ListNode forward = node.next.next;
                while (forward != null && node.next.val == forward.val) {
                    forward = forward.next;
                }
                node.next = forward; 
            } else {
                node = node.next;
            }
        }

        return dummy.next;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 最初に次のような解法を思いつく:
    - head ノードも除外の対象となる可能性があるので、まずは head を next に代入した dummy ノードを作る
    - dummy ノードと同じアドレスを指す node ノードを作り、これを使って次のように走査する:
        - node.next と node.next.next を比較し、値が同じである限り、 node.next に node.next.next の次の値を代入していく
        - node に node.next を代入する
        - node より先に2つ以上ノードがある限り上記を繰り返す
    - dummy ノードの next を 返す
- これを次のように実装する:

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) return null;

        ListNode dummy = new ListNode(0, head);
        ListNode node = dummy;
        while (node.next != null && node.next.next != null) {
            if (node.next.val == node.next.next.val) {
                ListNode forward = node.next.next;
                while (node.next.val == forward.val) {
                    forward = forward.next;
                }
                node.next = forward; 
            } else {
                node = node.next;
            }
        }

        return dummy.next;
    }
}
```

- Null pointer exception が発生したので、ちゃんと考えずに当てずっぽうで1つめの while の条件文の文頭に `node != null &&` を足してみる  
  (これは良くなかったと今は反省している。当てずっぽうなのに加え、エラーが出た行すらちゃんと見てない。)  
- 結果次の実装になって submit するものの、また同じ箇所で同じ exception が発生:

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode node = dummy;
        while (node != null && node.next != null && node.next.next != null) {
            if (node.next.val == node.next.next.val) {
                ListNode forward = node.next.next;
                while (node.next.val == forward.val) {
                    forward = forward.next;
                }
                node.next = forward; 
            } else {
                node = node.next;
            }
        }

        return dummy.next;
    }
}
```

- 今度はちゃんとエラーを追い、2つ目の while 文が exception の発生箇所であると認識する
- 発生原因が forward の null チェックを行わないことであると認識し、このステップ冒頭の実装に修正して submit したらパスした

## Step 2

表題の通り、次のように Step 1 で書いた実装を修正しました。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(0, head);
        ListNode prev = dummy;
        while (prev.next != null && prev.next.next != null) {
            ListNode current = prev.next, runner = prev.next.next;
            if (current.val == runner.val) {
                while (runner != null && current.val == runner.val) {
                    runner = runner.next;
                }
                prev.next = runner; 
            } else {
                prev = prev.next;
            }
        }

        return dummy.next;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 当てずっぽうで1つめの while 文に足した `node != null &&` だが、 while 文開始時点で node に代入されている dummy は null になることがなく、その後も prev は next 及び next.next が null にならない限り動かないため null にはなりえないので除外
- forward と変数に命名していたのが適当だったか考え直す:
    - このような3つのポインタを使った解法ではどのような命名が適当か ChatGPT に聞いたら、 prev, current, runner と返答
    - ChatGPT は集積された過去の解答の解答からもっともそれらしい値を返すものだと認識しており、実際過去に解いた問題の模範解答でも同じ命名がされていたことがあったのに加え、直感的にも良い命名だと感じたのでこの返答は採用に値すると判断
    - 以前 Discord で oda さんが「current という変数名には情報がほぼない」といったことを仰ってたが、この場合は「prev, runner に対応する、連続する走査のあるイテレーションにおいて基点となるノード」という意味合いがあると判断し、 current をそのまま採用
- 元々 prev, runner にあたるノードが変数化されていたので、可読性向上のため current にあたるものも変数化
- 上記全て踏まえて、このステップの最初に示した実装に修正した
- 処理の流れを言語化したところ次のようになり、考えられる限り素直な記述になったと判断したためここでステップ2は終了とした

```
3つのポインタを使って解きます。一つは宣言時に head の前に配置される prev、ほかは重複チェックに使う、 prev の一つ後ろに配置される current と、そのさらに一つ後ろに配置される runner です。  

まずは head を next に持つ dummy を宣言します。これは head ノードが削除対象になった際、それに代わって先頭になったノードを保持するのに使います。これを走査に使う prev に代入しましょう。  
prev の後ろに2つ以上ノードがある限り走査を続けます。ある走査において、 current と runner の指す値が等しい限り runner をひとつずつ前に進めます (runner が null でないかのチェックも合わせて必要になります)。走査の開始時に current と runner の指す値が等しくなければ prev をひとつ前に進めます。  
走査対象がなくなったら dummy.next を返して処理は終了です。
```

## Step 3

一回目の解答時、自分の実装に対して理解が浅い部分があり、そこについて考えていたら約11分かかってしまった。走査の終了時にするべき実装を Step 2 最下部で言語化した中で意識できていなかったので、あらたに `runner が指す値が current のものと別になったら、次の走査で今の runner とその先のノードを比較するために prev.next に runner を代入します。` という文言を上記文面に挿入した。  
修正後の文言は次の通り:

```
3つのポインタを使って解きます。一つは宣言時に head の前に配置される prev、ほかは重複チェックに使う、 prev の一つ後ろに配置される current と、そのさらに一つ後ろに配置される runner です。  

まずは head を next に持つ dummy を宣言します。これは head ノードが削除対象になった際、それに代わって先頭になったノードを保持するのに使います。これを走査に使う prev に代入しましょう。  
prev の後ろに2つ以上ノードがある限り走査を続けます。ある走査において、 current と runner の指す値が等しい限り runner をひとつずつ前に進めます (runner が null でないかのチェックも合わせて必要になります)。走査の開始時に current と runner の指す値が等しくなければ prev をひとつ前に進めます。  
走査対象がなくなったら dummy.next を返して処理は終了です。
```

まず Step 1 の時点で自分が何をしているのか把握しきれていないまま submit をした結果パスしたので満足し、 Step 2 が終わった時点でも理解の浅い点を認識できなかったことを反省。

その後一回あたり約3分で解答し終了した。

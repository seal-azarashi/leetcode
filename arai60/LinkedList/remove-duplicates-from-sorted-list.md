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

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 4 m 58 s
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

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- まず、次の解き方を思いついたのでループで実装を試みる:
    - 各ノードの val が、その次のノードの val と等しければ、さらにその次のノードに連結する (node.next に node.next.next を代入する)
    - もし node, node.next が null の場合、走査を終えて head を返す
- 実装は次のようになったが...

```java
// 一部テストケースで fail
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
- テストケース失敗の原因となった上記の欠点がなく、かつ一回の iteration で重複する要素を全て除くよう、次のように修正したが...

```java
// Null pointer exception 発生
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

表題の通り、次のように Step 1 で書いた実装を修正しました。

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

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 2つめの while 文に `node.next != null` があるが、これは null ~ を防ぐためにつけたものだった
- やりたかったのは「ここで node.next が null になったら (もう走査対象がないため) 処理を中断する」ことなので、これに沿うような形に変えてみる

```java
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
- 1つめの while 文の終了条件の書き方も変えたくなったので、冒頭に示したコードのように修正した
- 処理を口頭で説明するならば、次のようにするのが自然かなと考えるので、これで素直な処理になったように思う
    - 先頭のノードから順に、その次のノードとペアで処理していきます。末尾にたどりついたら head を返します。
    - 処理中、対象のノードの val が次のノードのものと等しかったら、さらにその次のノードを next に代入します。これを val と next が異なるものになるまで繰り返します。next が null になったらもう操作対象がないので、 head を return しましょう。

### 再帰で実装するパターン

再帰での実装パターンを追加しました。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        return deleteDuplicatesRecursively(head);
    }

    private ListNode deleteDuplicatesRecursively(ListNode node) {
        if (node == null || node.next == null) return node;
        node.next = deleteDuplicates(node.next);
        return node.val == node.next.val ? node.next : node;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- リストが大きいとスタックオーバーフローが発生するリスクがあるのはわかりつつ、再帰での実装も書けるようにしておこうと試みる
- 良い書き方を思いつかなかったので、 LeetCode の Solution にあった解法を参考に、処理を書く

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null || head.next == null) return head;
        head.next = deleteDuplicates(head.next);
        return head.val == head.next.val ? head.next : head;
    }
}
```

- 記述量が少ないのである種きれいにも見えるが、これだと走査対象のノードの名称が head であることに違和感を感じる
- 対象のノードは head かもしれないし tail かもしれないしそのどちらでもないかもしれないので、冒頭に示したコードのように修正した
- 末尾再帰最適化を試みようとするも、 Java で実現するには[どうにも大変そう](https://backpaper0.github.io/ghosts/optimized_tail_call_recursive_fibonacci_in_java.html#/)で時間内に書くような作業ではなさそうなので、今回も断念

## Step 3

Step 2 に書いたループと再帰のパターン両方で実施。どちらも大体2分弱で全 submission をミスなく完了。  
再帰の方を書いてる最中、1つめの deleteDuplicatesRecursively の if 文にある判定条件 `node == null` は、リストの数が0のときに対応出来るようにするもので、毎回の走査に必要ではないと気づき、以下のように修正した。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) return null;
        return deleteDuplicatesRecursively(head);
    }

    private ListNode deleteDuplicatesRecursively(ListNode node) {
        if (node.next == null) return node;
        node.next = deleteDuplicatesRecursively(node.next);
        return node.val == node.next.val ? node.next : node;
    }
}
```

これはループのパターンにも言えることなので、そちらも次のように修正した。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) return null;

        ListNode node = head;
        while (node.next != null) {
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

## Step 4

頂いた次のレビューを元に、ループのパターンについて、2つめの while 文を修正した: https://github.com/seal-azarashi/leetcode/pull/3#discussion_r1624688139

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) return null;

        ListNode node = head;
        while (node != null) {
            while (node.next != null && node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }

        return head;
    }
}
```

他にも上記の思考ログに書いてきたコードたちについてもレビューをもらい、指摘を取り入れている。詳しくは各レビューコメントを参照。

## Step 5

[ahayashi さんの記事](https://hayapenguin.com/notes/Posts/2024/04/24/how-to-practice-coding-effectively#%E5%85%B7%E4%BD%93%E7%9A%84%E3%81%AA%E7%B7%B4%E7%BF%92%E6%96%B9%E6%B3%95)にならい、3, 7, 30日後に再度解いていきます。

### 3日後の再チャレンジ

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode node = head;
        while (node != null) {
            while (node.next != null && node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }

        return head;
    }
}
```

メソッドの1行目に書いていた null チェックを書かなくなっていた。  
これは while 文の判定を `node.next != null` としていた時に記述したもので、その後のレビューを受け入れた結果不要になったもの。なのでより良い解答が出せたと考えて良さそう。

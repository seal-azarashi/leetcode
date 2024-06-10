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
- 直前に受けた模擬面接でされた関数名の指摘を思い出し、一部関数名も修正

```java
// Time Complexity: O(n), Space Complexity: O(1)
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;
        return detectCycleRecursively(head, head.next);
    }

    private boolean detectCycleRecursively(ListNode one, ListNode two) {
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

[oda さんの追加レビュー](https://discord.com/channels/1084280443945353267/1245404801177616394/1245560536335253504)を受けて:

- 再帰で処理を書くことについて:
    - 今回の処理を再帰で書くことの問題点を把握した:
        - Java について、 JVM 依存ではあるが[10年ほど前の投稿](https://stackoverflow.com/questions/20030120/what-is-the-default-stack-size-can-it-grow-how-does-it-work-with-garbage-colle)によると、スタックのサイズは最大でも1MB程度
            - `Xss` オプションつけてサイズを大きくする工夫がされることがままある
            - ちなみに [C/C++ だと多くて10MB程度](https://stackoverflow.com/questions/1825964/c-c-maximum-stack-size-of-program-on-mainstream-oses/45762715#45762715)
        - 従って再帰呼び出しが深くなるような処理において、Java の場合だと1万程度でスタックオーバーフローが発生すると考えられる
            - これはループで書く分には問題ない程度の量である
        - よって再帰で処理を書く場合はこれを考慮し、[短い方から再帰して、長い方は末尾再帰最適化する](https://discord.com/channels/1084280443945353267/1235829049511903273/1236256946403807323)よう実装するといった工夫が必要になる可能性がある
            - 末尾再帰最適化とは、末尾呼出しのコードを、戻り先を保存しないジャンプに変換することによって、スタックの累積を無くし、効率の向上などを図る手法である ([参考](https://ja.wikipedia.org/wiki/%E6%9C%AB%E5%B0%BE%E5%86%8D%E5%B8%B0#%E6%9C%AB%E5%B0%BE%E5%91%BC%E5%87%BA%E3%81%97%E6%9C%80%E9%81%A9%E5%8C%96))
                - 関数の最終処理が純粋に自身の関数を呼ぶだけにする書き方を末尾呼び出しと呼ぶ
                - 関数型言語等と異なり、 Java は末尾呼び出しをしても末尾再帰最適化が自動で行われない
                - よく読んでないけど、[この記事](https://www.k-pmpstudy.com/entry/2020/10/24/TCO)によると Java で末尾再帰最適化を実装するのは大変そう
    - 解法を選択する際は上記考慮に入れるべきだと認識した
    - 一方でループと再帰の行き来ができることは期待されているので、再帰がベストでないと考えられる問題でも、両方のアプローチで書けるように練習するのは今後も継続する
- また while については一様でない色々な書き方が出来るので、これを使ったアプローチを思いついた場合でも、他にどのような書き方が出来るのかは考えるようにしたい
- 初めて使うクラス等についてはリファレンスに目を通すようにとのアドバイスもあった
    - 今回の対象はこれら:
        - https://docs.oracle.com/javase/8/docs/api/java/util/Set.html
        - https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html 
    - 前から見るようにしていたので今後もやっていく
    - もっと言うとわざわざノートにまとめるなんてこともやってたが、そこまでやる必要はないよな、と思い返した

その後 nodchip さんから次のレビューを頂く。

> recursive() という関数名に違和感を感じました。関数名には、関数が行う処理の内容を端的に表した英単語・英語句を付けることをお勧めします。また、関数名は命令形の単語または英語句とすることをお勧めします。
> https://google.github.io/styleguide/javaguide.html#s5.2.3-method-names

添えられている Google Java Style Guide を読み、関数名を `isCycleDetected` に修正した。

```java
// Time Complexity: O(n), Space Complexity: O(1)
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;
        return isCycleDetected(head, head.next);
    }

    private boolean isCycleDetected(ListNode one, ListNode two) {
        if (one == null || two == null || two.next == null) return false;
        if (one == two) return true;
        return isCycleDetected(one.next, two.next.next);
    }
}
```

上記 Guide には "Method names are typically verbs or verb phrases" とあるものの、今回返り値の型が boolean なので、 prefix に `is` をつけた形にした ([参考](https://rules.sonarsource.com/java/tag/convention/RSPEC-2047/))。

## Step 5

### 二度目の解答

#### HashSet を用いた解答

```java
// Time taken: 2 m 11 s
public class Solution {
    public boolean hasCycle(ListNode head) {
        Set<ListNode> visitedNodes = new HashSet<>();
        ListNode node = head;
        while (node != null) {
            if (visitedNodes.contains(node)) return true;
            visitedNodes.add(node);
            node = node.next;
        }

        return false;
    }
}
```

前回とほぼ一緒だが、 HashSet の宣言時に変数名を意味のあるものにしている。  
レビューで以前指摘された内容だったので、それを引き出せてよかった。

#### ループを用いた Floyd's ~ の解法

```java
// Time taken: 2 m 13 s
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }

        return false;
    }
}
```

slow と fast をワンライナーで定義した以外は同じ。

#### 再帰を用いた Floyd's ~ の解法

```java
// Time taken: 6 m 49 s
public class Solution {
    public boolean hasCycle(ListNode head) {
        return detectCycleRecursively(head, head);
    }

    private boolean detectCycleRecursively(ListNode slow, ListNode fast) {
        if (fast == null || fast.next == null) return false;
        
        slow = slow.next;
        fast = fast.next.next;
        return slow == fast ? true : detectCycleRecursively(slow, fast);
    }
}
```

再帰関数の引数名が one, two から slow, fast になって読みやすくなった気がする。  
不要だった slow ポインタのチェックをしなくなっていた。  

### 三度目の解答

HashSet、ループの解法はスラスラ解けたが再帰の解法を書くのに少々時間がかかった。

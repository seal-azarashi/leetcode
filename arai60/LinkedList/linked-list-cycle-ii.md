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
// Time Complexity: O(n), Space Complexity: O(n)
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
// Time Complexity: O(n), Space Complexity: O(1)
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
// Time Complexity: O(n), Space Complexity: O(1)
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
// Time Complexity: O(n), Space Complexity: O(n)
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

- こちらの問題を解いた後、別のプルリクエストで受けたレビューを元に、上に挙げた3つの解法をそれぞれ次のように書き直した

```java
// Time Complexity: O(n), Space Complexity: O(n)
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
// Time Complexity: O(n), Space Complexity: O(1)
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
// Time Complexity: O(n), Space Complexity: O(n)
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

HashSet を使ったループの解法について、返り値は null で良いと指摘があったので修正。

```java
// Time Complexity: O(n), Space Complexity: O(n)
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> set = new HashSet<>();
        while (head != null) {
            if (set.contains(head)) return head;
            set.add(head);
            head = head.next;
        }

        return null;
    }
}
```

ここまで書いた後、 oda さんから[追加レビュー](https://github.com/seal-azarashi/leetcode/pull/2/files#r1621223636)が来た。これにある通り、最初の while 文のあとに、実質その終了条件と同じことを書いていることに違和感を覚える。

> いろいろな選択肢が見えたうえで、これを選んでいるならばいいのですが、そうでないならば、いろいろな選択肢があることを見ておいてください。
> https://discord.com/channels/1084280443945353267/1221030192609493053/122567490144528386

いろいろな選択肢を見たうえで選んでいなかったのを反省。添えられたリンクを読む。  
これを踏まえ、この違和感、素直じゃない感じはどこから来るのかを考えていたら、上記リンクに引用される「[機械の使い方の説明](https://discord.com/channels/1084280443945353267/1192728121644945439/1194203372115464272)」を思い出した。

> 「機械の使い方の説明です。まず、青いランプが5つついていることを確認してください。ランプがついていなかった場合は、直ちに使用を中止して事務所に連絡してください。…長い使い方の説明…。」  
> のほうが素直ですよね。

今回実装していた処理を日本語で説明するとして、上記と同じ順序で述べるなら次のようになるだろうか。

```
まず、 Floyd's ~ を使ってサイクルを探しましょう。見つからない場合は、 null を返します。
slow, fast ポインタが出会ったノードと head ノードから、それぞれ1ずつポインタを進め、両者が出会うノードを返してください。
```

翻って、今の実装を説明すると次のようになるだろうか。

```
まず、 Floyd's ~ を使ってサイクルを探しましょう。 "サイクルを探す処理が終わったら、サイクルが見つかったのかどうか確認しましょう。" 見つからない場合は、 null を返します。
slow, fast ポインタが出会ったノードと head ノードから、それぞれ1ずつポインタを進め、両者が出会うノードを返してください。
```

このダブルクオーテーションで囲った部分に素直じゃない感じがあるのではと思い、コードを次のように修正する。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (true) {
            if (fast == null || fast.next == null) return null;

            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) break;
        }

        while (head != slow) {
            head = head.next;
            slow = slow.next;
        }

        return head;
    }
}
```

これで上記太字部分がなくなり、素直になったように感じた。  

さて、 [oda さんが添えていたリンク](https://discord.com/channels/1084280443945353267/1221030192609493053/1225674901445283860)では、「こういう構造のときには、Y を外に出したいです。いくつか手があります」とした上で、他の選択肢も挙げられていた。

```
> 1.関数化。  
> while A:  
>     短いX  
>     if B:  
>         Y()  
>         return C  
> 短いZ  
> return D
```

これを Java で書くとこんな感じだろうか。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return findWhereTheCycleStarts(head, slow);
        }

        return null;
    }

    private ListNode findWhereTheCycleStarts(ListNode head, ListNode node) {
        while (head != node) {
            head = head.next;
            node = node.next;
        }

        return head;
    }
}
```

この処理を日本語で書いてみるとこんな感じだろうか。

```
まず、 Floyd's ~ を使ってサイクルを探しましょう。slow, fast ポインタが出会ったノードと head ノードから、それぞれ1ずつポインタを進め、両者が出会うノードを返してください。
見つからない場合は、 null を返します。
```

最初の修正案の方の日本語と比べると、若干素直さに欠けるような気がする。[oda さんの添えた](https://discord.com/channels/1084280443945353267/1192728121644945439/1194203372115464272)リンクにあった、比較的素直じゃない例として挙げられた次の文言に似ている。  
「機械の使い方の説明です。まず、青いランプが5つついていることを確認してください。ついている場合、…使い方の説明…。ランプがついていなかった場合は、直ちに使用を中止して事務所に連絡してください。…機械の使い方の続き…。」

次の選択肢を見よう。

```
> Python 独特の文法。
> while A:
>     短いX
>     if B:
>         break
> else:
>     短いZ
>     return D
> 長いY
> return C
```

自分は Java で書くのでこれは一旦考慮から外すが、レビューを受ける前の解法に似ている気がする。

次の選択肢はどうか。

```
> 無限ループを使う。
> while 1:
>   if not A:
>       短いZ
>       return D
>   短いX
>   if B:
>       break
> 長いY
> return C
```

これは最初の修正案とほぼ同じだろうか。

最後の選択肢を見る。

```
> 関数化
> def F():
>     while A:
>         短いX
>         if B:
>             return True
>     return False
> 
> if not F():
>     短いZ
>     return D
> 長いY
```

Java で書くとこんな感じだろうか。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode whereSlowAndFastMeet = findWhereSlowAndFastMeet(head);
        if (whereSlowAndFastMeet == null) return null;

        while (head != whereSlowAndFastMeet) {
            head = head.next;
            whereSlowAndFastMeet = whereSlowAndFastMeet.next;
        }
        return head;
    }

    private ListNode findWhereSlowAndFastMeet(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return slow;
        }

        return null;
    }
}
```

切り出した関数の返り値を boolean にするとどうしても処理が冗長になるので、少しアレンジを加えている。  
日本語にすると次のようになるので、レビューを受けた解法と同じになってしまったか。

```
まず、 Floyd's ~ を使ってサイクルを探しましょう。 "サイクルを探す処理が終わったら、サイクルが見つかったのかどうか確認しましょう。" 見つからない場合は、 null を返します。
slow, fast ポインタが出会ったノードと head ノードから、それぞれ1ずつポインタを進め、両者が出会うノードを返してください。
```

さて、これでいろいろな選択肢を洗い出せただろうか。  
最初に出した修正案がベストかと思うので、レビューに対する解答としてこれを提示したい。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (true) {
            if (fast == null || fast.next == null) return null;

            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) break;
        }

        while (head != slow) {
            head = head.next;
            slow = slow.next;
        }

        return head;
    }
}
```

ただ、最初の while 文内の null チェックが末尾に来たほうが良さそうだなとおもっている。しかしながら null pointer exception を回避しつつそのように書く方法が思いつかなかった。その点はまだ気持ち悪いので、いいやりかたを思いついたらアプライしたい。

その後、 hayashi-ay さんから次のレビューが来た。

> 変数名はもう少しこだわっても良いかなと思います。犬に犬と名前をつけてる感覚です。seenNodesとかvisitedNodesとかでしょうか？

仰る通りなので、頂いた候補の中から自分のイメージにより合致する `visitedNodes` を採用させて頂いた。加えて、変数名だけでなく関数名も `recursive` もよりこだわった方がいいと考え、 `findThBeginningOfCycle` 修正した。

```java
// Time Complexity: O(n), Space Complexity: O(n)
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> visitedNodes = new HashSet<>();
        return findThBeginningOfCycle(visitedNodes, head);
    }

    private ListNode findThBeginningOfCycle(Set<ListNode> visitedNodes, ListNode node) {
        if (node == null) return null;
        if (visitedNodes.contains(node)) return node;
        visitedNodes.add(node);
        return findThBeginningOfCycle(visitedNodes, node.next);
    }
}
```

```java
// Time Complexity: O(n), Space Complexity: O(1)
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
// Time Complexity: O(n), Space Complexity: O(n)
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> visitedNodes = new HashSet<>();
        while (head != null) {
            if (visitedNodes.contains(head)) return head;
            visitedNodes.add(head);
            head = head.next;
        }

        return null;
    }
}
```

## Step 5

[ahayashi さんの記事](https://hayapenguin.com/notes/Posts/2024/04/24/how-to-practice-coding-effectively#%E5%85%B7%E4%BD%93%E7%9A%84%E3%81%AA%E7%B7%B4%E7%BF%92%E6%96%B9%E6%B3%95)にならい、3, 7, 30日後に再度解いていきます。

### 3日後の再チャレンジ

#### HashSet を用いたパターン

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        Set<ListNode> visitedNodes = new HashSet<>();
        ListNode node = head;
        while (node != null) {
            if (visitedNodes.contains(node)) return node;
            visitedNodes.add(node);
            node = node.next;
        }

        return null;
    }
}
```

- 全く同じ答えがでた

#### Floyd's ~ の応用パターン

```java
// Time taken: 7 m 12 s
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return findCycleStart(head, slow);
        }

        return null;
    }

    private ListNode findCycleStart(ListNode head, ListNode meetingPoint) {
        while (true) {
            if (head == meetingPoint) return head;
            head = head.next;
            meetingPoint = meetingPoint.next;
        }
    }
}
```

- 関数名を `findCycleStart` とより短いものにした
- その関数の第二引数を、 `node` ではなく `meetingPoint` に変え、 slow と fast ポインタが会う地点のノードであることを示すようにした

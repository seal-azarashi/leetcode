# 49. Group Anagrams

LeetCode URL: https://leetcode.com/problems/group-anagrams/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 解いた時間: 50分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n)
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<List<Integer>, List<String>> wordDistributionMap = new HashMap<>();
        for (String str : strs) {
            List<Integer> wordDistribution = getWordDistribution(str);
            List<String> wordList = wordDistributionMap.getOrDefault(wordDistribution, new LinkedList<>());
            wordList.add(str);
            wordDistributionMap.put(wordDistribution, wordList);
        }

        List<List<String>> result = new LinkedList<>();
        for (List<String> strList : wordDistributionMap.values()) {
            result.add(strList);
        }
        return result;
    }

    private List<Integer> getWordDistribution(String str) {
        List<Integer> wordDistribution = new ArrayList<>(Collections.nCopies(26, 0));
        for (char character : str.toCharArray()) {
            int count = wordDistribution.get(character - 'a');
            wordDistribution.set(character - 'a', count + 1);
        }
        return wordDistribution;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- Constraints に "strs[i] consists of lowercase English letters." とあるのを読み、最初に次の解法を思いつく: 
    - アルファベットと同じ要素数の配列を用意し、文字列の全文字を対象に、各インデックスに対応する (対象の char 値 - 'a' をインデックス値とする) 文字の数を要素として配列に格納する
    - HashMap に、上記の配列を Key、同じ配列を生成できる文字列たちを格納するリストを Value として格納する
    - HashMap の各 value を一つのリストに入れて返す
- 面接で話すなら、もっと自然に思いつきそうな、文字をソートして key にするアプローチから検討を始めるかも
- 一度は strs の各要素を走査する必要があるはずと考え、そうであれば、O(n) の時間計算量となる上記の解法は悪くないと考え解答を書きはじめる
- 要素へのランダムアクセスが必要な場合は ArrayList, 末尾への要素追加しかしないならよりオーバーヘッドの少ない LinkedList を使おうと考える
- 知識が曖昧な箇所をググりながらコードを書く:
    - アルファベットの数って26で合ってたっけ (面接ではググらずにしれっと頭の中で数えたほうがいいか)
    - HashMap の value だけを返すメソッドはなんだったか
        - values() だった
        - entrySet(), keySet() と関連性がなさそうな名前で想起しづらいな (まあでも set じゃないからしょうがない)
    - HashMap は Key に配列を入れた際、同値判定が期待通りに動くのか？
        - 内部的に equals() をつかっており、配列同士の比較はオブジェクトの参照先アドレスの比較となるため動かない
            - 実際動かして、上記正しいことは確認できたが、いつもの Oracle のドキュメントにこれに関して記述が見当たらない
            - なんかカーネギーメロン大の Confluence に書いてあるのを見つけた: https://wiki.sei.cmu.edu/confluence/display/java/EXP02-J.%2BDo%2Bnot%2Buse%2Bthe%2BObject.equals%28%29%2Bmethod%2Bto%2Bcompare%2Btwo%2Barrays
            - ちなみに比較を期待通りに行うこと自体は Arrays.equals() を使えばできる: https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#equals-boolean:A-boolean:A-
        - List クラスなら期待通りに動作するので、こちらを代わりに使う: https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/List.html#equals-java.lang.Object-
    - ArrayList はサイズを引数に渡して宣言した際、配列とは異なり同数のゼロ値を持った要素を用意してはくれないので、その処理を簡潔に実装するのに使える Collections.nCopies() メソッドを見つける: https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#nCopies-int-T-
- List の set() メソッドを書くべきところに add() を書いてしまっていたため期待通りでない挙動になり、デバッグにちょっと手間取った

## Step 2

Step 1 の実装を次のように修正しました。 

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<List<Integer>, List<String>> charFrequencyMap = new HashMap<>();
        for (String str : strs) {
            List<Integer> charFrequency = new ArrayList<>(Collections.nCopies(26, 0));
            for (char character : str.toCharArray()) {
                int charIndex = character - 'a';
                charFrequency.set(charIndex, charFrequency.get(charIndex) + 1);
            }
            charFrequencyMap.computeIfAbsent(charFrequency, _ -> new LinkedList<>()).add(str);
        }
        
        return new LinkedList<>(charFrequencyMap.values());
    }
}
```

- distribution より frequency の方が一般的みたいだったので書く変数を修正
- index を算出する `character - 'a'` が2箇所あったので、デバッグを用意にするためこれを変数化
- computeIfAbsent の代わりに putIfAbsent を使うのはどうかと見てみたが、指定の key がマッピングされていない場合 (この操作によって値がマッピングされるにもかかわらず) 返り値が null になるのが直感的でないと感じた
- ラムダ式を用いるため少しトリッキーに感じるが、computeIfAbsent だとマッピングされた値が返るために add() メソッドをチェーン呼び出しできるのを利点に感じ、今回はこちらを採用した
    - メソッドチェーン呼び出しをする上でいくつか懸念点があったが、今回の実装は非常にシンプルなものなので、これらは無視できる程度だと判断:
        - チェーンの各ステップで一時的な中間オブジェクトが生成されるはずなので、その分のオーバーヘッドが処理時間を有意に遅くしないだろうか
        - 中間メソッドのデバッグが必要になったら、その実施が大変にならないだろうか

## Step 3

Step 2 と全く同じコードを書きました。  
そのまま書くだけなら大体6分ぐらい、本番の面接を想定して英語で思考を説明しながら書くと8分ぐらいかかりました。


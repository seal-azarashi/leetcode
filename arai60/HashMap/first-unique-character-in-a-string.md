# 387. First Unique Character in a String

LeetCode URL: https://leetcode.com/problems/first-unique-character-in-a-string/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 解いた時間: 15分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n) 
class Solution {
    public int firstUniqChar(String s) {
        // サロゲートペアや合成文字等、char の値を複数用いないと表現出来ない文字が来ないことを想定しています。
        char[] charArray = s.toCharArray();
        Set<Character> foundChars = new HashSet<>();
        Set<Character> duplicates = new HashSet<>();
        for (char c : charArray) {
            if (foundChars.contains(c)) {
                duplicates.add(c);
                continue;
            }
            foundChars.add(c);
        }
        for (int i = 0; i < charArray.length; i++) {
            if (!duplicates.contains(charArray[i])) {
                return i;
            }
        }

        return -1;
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 次の解法を思いつく:
    1. s を一文字ずつ先頭から見ていって、走査済の文字を set に入れつつ、既に set にある (重複している) 文字を別の set に入れる。
    2. 再度 s を一文字ずつ先頭から見ていって、重複文字の set に含まれないものをみつけたら、そのインデックスを返す
- TC, SC ともに O(n) なので悪くはない
- 他に出来るアプローチも考える:
    - 文字列をソートして、隣に同じ要素がない (ユニークである) 要素を返す?
        - -> 順序がわからなくなるので🙅
    - "s consists of only lowercase English letters" とあるので、26の配列の、char - 'a' に対応する要素に値を格納していく？
        - -> 見つかっていないもの、重複してないものの情報は必要ないから set を使うアプローチの方がいい
    - s を一度だけ走査して済ませられないか？
        - -> 時間内には思いつかなそうなのでひとまず見送り
- どんなユースケースがあるだろうか
    - "s consists of only lowercase English letters" とあるので、限られたデータセットから、何かしらの分析的処理をする際の中間処理として用いられる？
    - データセットが限られるとはいえ、サロゲートペアのような値が入らないとは限らないから、必要であれば処理を変える必要がある、という旨は話したい
        - しかしそれがそらで書けるかと言うと厳しい... Java のドキュメント見ながら書くことになってしまいそう
- 考慮したユースケースを加味してコメントを書く

## Step 2

### HashMap を使う解法

他の方の解法を見てて、そもそも HashMap のカテゴリの問題だったことを思い出して書きました。こちらの方が全然見通しいいですね。

```java
// 時間計算量: O(n)
// 空間計算量: O(n) 
class Solution {
    private static final int NOT_FOUND = -1;

    public int firstUniqChar(String s) {
        Map<Character, Integer> characterFrequency = new HashMap<>();
        for (char c : s.toCharArray()) {
            int frequency = characterFrequency.getOrDefault(c, 0);
            characterFrequency.put(c, frequency + 1);
        }
        for (int i = 0; i < s.length(); i++) {
            if (characterFrequency.get(s.charAt(i)) == 1) {
                return i;
            }
        }

        return NOT_FOUND;
    }
}
```

- [iwasa さんのレビュー](https://github.com/kazukiii/leetcode/pull/16/files#r1650394921)をみて、定数 `NOT_FOUND` を追加

### 要素数26の配列を使う解法

```java
// 時間計算量: O(n)
// 空間計算量: O(1) 
class Solution {
    private final int NOT_FOUND = -1;
    private final char ALPHABET_OFFSET = 'a';
    private final int ALPHABET_SIZE = 26;

    public int firstUniqChar(String s) {
        int[] alphabetFrequency = new int[ALPHABET_SIZE];
        for (char c : s.toCharArray()) {
            alphabetFrequency[c - ALPHABET_OFFSET]++;
        }
        for(int i = 0; i < s.length(); i++) {
            if (alphabetFrequency[s.charAt(i) - ALPHABET_OFFSET] == 1) {
                return i;
            }
        }

        return NOT_FOUND;
    }
}
```

- 最初の解法と同じく計算量 O(n) なので step 1 では使わなかったが、LeetCode 上では実行速度が約5倍に向上していて面白かった
- 【🚨レビュワーの皆様、この点何か思いつくものなどありましたらご指摘頂けると嬉しいです】この処理が早い要因としては次のようなものが考えられるだろうか:
    - 要素数26の配列である alphabetFrequency はオーバーヘッドを含めなければ100バイト程度の大きさしかないので、これを扱う際には、L1 データキャッシュが活用されていると思われる
    - ハッシュ計算、ハッシュ衝突時の走査といった処理がなくなり、代わりにずっと単純な `s.charAt(i) - 'a'` が実行されるようになった
    - 単純になったので多分 JVM の JIT コンパイラにとってもより最適化しやすくなっているかもしれない
    - boxing/unboxing によるオーバーヘッドがなくなった (from [kazukiii さんのレビュー](https://github.com/seal-azarashi/leetcode/pull/15#discussion_r1704748299))
- そういえば空間計算量は O(1) に改善されるので、最初からこちらを選んでもよかったなと振り返って思う
    - 一方で lower case english letter 以外は扱えない、ユースケースを想像すると中々ピーキーな実装になるので、面接官に対しては、過度なチューニングをいきなりし始めるような印象を与えないように気をつけたい
- 定数を static にする利点がほぼないので、この修飾子を外した

### その他メモ

- 順序を持つ LinkedHashMap を使う案があったが、走査回数は変わらない & 2回目の走査を行う for 文の記述に情報量が増え、見づらくなる印象があったので採用を見送った
- [平衡木を使うアプローチ](https://github.com/nittoco/leetcode/pull/20#discussion_r1642843424)もあるが、この問題に関しては step 1, 2 に記載の解法よりも計算量において劣るため採用を見送った

## Step 3

```java
// 時間計算量: O(n)
// 空間計算量: O(1) 
class Solution {
    private final int ALPHABET_SIZE = 26;
    private final char ALPHABET_OFFSET = 'a';
    private final int NOT_FOUND = -1;

    public int firstUniqChar(String s) {
        int[] alphabetFrequency = new int[ALPHABET_SIZE];
        for (char c : s.toCharArray()) {
            alphabetFrequency[c - ALPHABET_OFFSET]++;
        }
        for (int i = 0; i < s.length(); i++) {
            if (alphabetFrequency[s.charAt(i) - ALPHABET_OFFSET] == 1) {
                return i;
            }
        }

        return NOT_FOUND;
    }
}
```

- Step 2 の要素数26の配列を使う解法とほぼ同じ
- 定数の並び順だけ、firstUniqChar() で登場する順に変わった
    - 特に読みやすさに変化があるとは思えないが、書いていて自然とこうなったので、書く際に脳に負荷がかかりづらい順番になったのかなという印象

## Step 4

```java
// 時間計算量: O(n)
// 空間計算量: O(1) 
class Solution {
    private static final int ALPHABET_SIZE = 26;
    private static final char ALPHABET_OFFSET = 'a';
    public static final int NOT_FOUND = -1;

    public int firstUniqChar(String s) {
        int[] alphabetFrequency = new int[ALPHABET_SIZE];
        for (char c : s.toCharArray()) {
            alphabetFrequency[c - ALPHABET_OFFSET]++;
        }
        for (int i = 0; i < s.length(); i++) {
            if (alphabetFrequency[s.charAt(i) - ALPHABET_OFFSET] == 1) {
                return i;
            }
        }

        return NOT_FOUND;
    }
}
```

- [fhiyo さんの指摘](https://github.com/seal-azarashi/leetcode/pull/15#discussion_r1704435525)を受け、firstUniqChar() の呼び出し元等から参照される可能性のある NOT_FOUND の修飾子を `public static final` に、また他のメンバも static になるよう修正した

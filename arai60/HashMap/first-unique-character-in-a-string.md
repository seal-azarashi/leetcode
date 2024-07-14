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
        // サロゲートペア等、char の値を複数用いないと表現出来ない文字が来ないことを想定しています。
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

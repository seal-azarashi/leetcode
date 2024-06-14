# 20. Valid Parentheses

LeetCode URL: https://leetcode.com/problems/valid-parentheses/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 実装にかかった時間: 11 m 50 s
class Solution {
    public boolean isValid(String s) {
        ArrayDeque<Character> stack = new ArrayDeque<>();
        for (int i = 0; i < s.length(); i++) {
            if (isLeftBracket(s.charAt(i))) {
                stack.push(s.charAt(i));
                continue;
            }

            if (stack.isEmpty()) return false;

            char leftBracket = stack.pop();
            char rightBracket = s.charAt(i);
            if (!isBracketsBalanced(leftBracket, rightBracket)) return false;
        }

        return stack.isEmpty();
    }

    private boolean isLeftBracket(char bracket) {
        return bracket == '(' || bracket == '{' || bracket == '[';
    }

    private boolean isBracketsBalanced(char leftBracket, char rightBracket) {
        return
            (leftBracket == '(' && rightBracket == ')') ||
            (leftBracket == '{' && rightBracket == '}') ||
            (leftBracket == '[' && rightBracket == ']');
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 結構前に自力で解けたはずだが今回は解法思いつかず、解答を見る  
  (めちゃくちゃ眠い時に解いたので脳が動いてくれなかったのかもしれないが、そもそもこの会でやってるみたいにちゃんと解いてなかったから記憶に定着してないのだ、だからしっかり取り組もうと思った)
- 解答見てどう解いたかを思い出して実装方針を考える
- まず、文字列内にある括弧の組み合わせが正しいかどうかを判定するのにおいて、基本的な考え方は次の通りだと整理する:
    1. 右括弧が現れたとき、その直前には対応する左括弧があるはず
    2. 2回連続で右括弧が現れたとき、その右括弧は、最後に現れた左括弧から数えて2つ目の左括弧に対応するはず
    3. 連続で現れる回数が代わっても 2 の規則性は変わらない
- 上記の考えを実装するにあたり、左括弧を stack に格納するのが適当であると考る
- まずは左括弧を格納するための Stack として変数 stack を宣言 ([参考](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html))
    - 些末な話ではあるが ArrayDeque のパフォーマンスが良いのを知っていたのでそれを採用 ([公式ドキュメントの冒頭を参考](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html))
- 左括弧であるかどうかの判定のため、メソッド isLeftBracket を宣言
    - 一度しか使われないものの、何をしているのかメソッド名が簡潔に表してくれるため、可読性が向上することを期待
- 対象の括弧同士がマッチするかどうかの判定のためのメソッド isBracketsBalanced を宣言
    - 同じく一度しか使われないものの、何をしているのかメソッド名が簡潔に表してくれるため、可読性が向上することを期待
    - 英語で「括弧の組み合わせがマッチしている」をなんと言うのか分からなかったので、ググって "balanced" という単語を見つけた (これは Step 2 でやるべきだった)
- 上記2つのメソッド、記述が長く書くのに時間がかかってしまい、どうにかもっと簡潔に出来ないものか... と思いつつもひとまず書き上げる
- 変数 leftBracket と rightBracket については無くても処理の実装は出来るが、次のように一行で書くと少し読みづらく感じるため残した: `if (bracketMap.get(leftBracketStack.pop()) != bracket) return false;`
    - 引数としてメソッドの呼び出しが渡されている
    - bracket に代入されている値が右括弧であることが、その行を見るだけでは読み取れない
- 最初の実装では、右括弧の数が左括弧の数を超えたときの想定が出来ておらず失敗
- 2度目の submit でパスした

## Step 2

Step 1 で作成した解答の修正を行いました。

```java
class Solution {
    public boolean isValid(String s) {
        // Key: left bracket, Value: right bracket
        Map<Character, Character> bracketMap = new HashMap<>();
        bracketMap.put('(', ')');
        bracketMap.put('{', '}');
        bracketMap.put('[', ']');

        ArrayDeque<Character> leftBracketStack = new ArrayDeque<>();

        for (char bracket : s.toCharArray()) {
            if (bracketMap.containsKey(bracket)) {
                leftBracketStack.push(bracket);
                continue;
            }

            if (leftBracketStack.isEmpty()) return false;

            char leftBracket = leftBracketStack.pop();
            char rightBracket = bracket;
            if (bracketMap.get(leftBracket) != rightBracket) return false;
        }

        return leftBracketStack.isEmpty();
    }
}
```

可読性向上のため、次の修正を実施しています:

- ループを拡張 for 文にし、走査中の char 値を bracket と命名
- 左括弧を格納するスタックの名称を stack から leftBracketStack に変更
- 2つの private メソッドを消去し、 HashMap を用いて括弧の対応関係を表現

## Step 3

一回目は5分半程度かかり、回を重ねるごとに早くなって三回目は4分半程度で実装できた。

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
- 最初の実装では、右括弧の数が左括弧の数を超えたときの想定が出来ておらず失敗
- 2度目の submit でパスした

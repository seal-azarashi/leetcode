# 929. Unique Email Addresses

LeetCode URL: https://leetcode.com/problems/unique-email-addresses/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

このステップで作成した実装は次の通りです。

```java
// 解いた時間: 20分ぐらい
// 時間計算量: O(n)
// 空間計算量: O(n) 
class Solution {
    public int numUniqueEmails(String[] emails) {
        Set<String> uniqueEmails = new HashSet<>();
        for (String email : emails) {
            StringBuilder stringBuilder = new StringBuilder();
            boolean isSkipping = false;
            boolean isAfterAtMark = false;
            for (char character : email.toCharArray()) {
                if (isAfterAtMark) {
                    stringBuilder.append(character);
                    continue;
                }

                if (character == '+') {
                    isSkipping = true;
                } else if (character == '@') {
                    isSkipping = false;
                    isAfterAtMark = true;
                }
                if (isSkipping || character == '.') {
                    continue;
                }
                stringBuilder.append(character);
            }

            uniqueEmails.add(stringBuilder.toString());
        }

        return uniqueEmails.size();
    }
}
```

以下、上記が完成するまでの試行錯誤の過程を、時系列順に思考ログとして書いていきます:

- 例外ハンドリングが複雑になりそうと思いきや、constraints がしっかりしているので、実装は割と単純に出来そう (Easy だし)
    - 面接ではどこまでバリデーションして欲しいか面接官に聞くと思う (仕事でフルスクラッチで実装するなら RFC 見たりするけど、ここではどこまでが求められてます？みたいな感じで)
- local name を操作して、結果を set に入れ、そのサイズを返すようにしたい
- String は immutable なので、StringBuilder を使う
- @ で split して local name だけ抜き出して操作をする案を最初に思いついたけど、constraints を見る限り、頭から一文字ずつ操作をしていけば行けそう
- 嫁とネトフリのボーイフレンド見ながら書いてて集中出来てなかったのか、一連の if, else if 文の構成に結構右往左往
    - @ に到達したら、その後はスキップ判定をしないようにすればきれいに書けた
    - `isSkipping || character == '.'` を else if に入れたら必要な isSkipping チェックをしなくなってしまってケース通らなくなったのを反省
    - else if があれば余計な判定を減らせるけど、それは微々たるパフォーマンスの向上にしかならないので、まずは if 文だけで実装を完了させるようにすればよかったかも

## Step 2

### Step 1 のアップグレード

```java
class Solution {
    public int numUniqueEmails(String[] emails) {
        Set<String> uniqueEmails = new HashSet<>();
        for (String email : emails) {
            // Constraints を侵害しない値のみ来ることを想定
            String[] emailComponents = email.split("@");
            StringBuilder sb = new StringBuilder();
            for (char c : emailComponents[0].toCharArray()) {
                if (c == '+') {
                    break;
                }
                if (c == '.') {
                    continue;
                }
                sb.append(c);
            }
            sb.append('@').append(emailComponents[1]);

            uniqueEmails.add(sb.toString());
        }

        return uniqueEmails.size();
    }
}
```

- local 部分にだけ着目すれば処理が簡潔に出来る
    - Yoshiki-iwasa さんの実装を参考にした: https://github.com/Yoshiki-Iwasa/Arai60/blob/db7c57a810f86268bf5c1d60bebac586f0fefe2d/problems/src/unique_email_addresses/step3.rs
    - emailComponents を宣言することになるが、それによって使用されるメモリ量の増加を、可読性の向上のメリットが上回ると判断
    - あとそもそも @ 以降はそのまま append すればよく、char array にしてひとつずつ走査する必要はないので、無駄に計算量が増えていた
- ローカル変数の割に冗長だった名前のものを、慣習に沿った sb と c に修正
- email のバリデーションが含まれないことに違和感を感じるので、Constraints を侵害しない値のみ来ることを想定していると追記

### Stream API の使用

```java
class Solution {
    public int numUniqueEmails(String[] emails) {
        return (int) Arrays.stream(emails)
            .map(this::normalizeEmail)
            .distinct()
            .count();
    }

    private String normalizeEmail(String email) {
        // Constraints を侵害しない値のみ来ることを想定
        String[] emailComponents = email.split("@");
        StringBuilder sb = new StringBuilder();
        for (char c : emailComponents[0].toCharArray()) {
            if (c == '+') {
                break;
            }
            if (c != '.') {
                sb.append(c);
            }
        }
        return sb.append('@').append(emailComponents[1]).toString();
    }
}
```

- 読みやすい
- この場合ボクシングも必要ないので、そのオーバーヘッドがない
- 並列化が必要な際は .parallel() をチェイニングするだけで実装だけは完了するのが素晴らしい
    - normalizeEmail が複雑になる場合は並列化したい... がまあこれに追加するにしても RFC に沿ってバリデーションするぐらいだからここではあまり効果を発揮しないかも
- シンプルなのでデバッグしづらいとかもなさそう
- ただ面接で書けって言われてスラスラは出てこないな... 一応書けるように訓練しておいた方がいいのだろうか？

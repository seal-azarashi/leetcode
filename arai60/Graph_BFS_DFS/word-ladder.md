# 127. Word Ladder

LeetCode URL: https://leetcode.com/problems/word-ladder/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

### 答えを見て解いたときの実装 (パターンと文字列の対応を用いた実装)

```java
// 時間計算量: O(N^2)
// 空間計算量: O(N)
class Solution {
    private static final char MATCHES_ANY_SINGLE_CHARACTER = '?';

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        wordList.add(beginWord);
        Map<String, List<String>> globWordsMap = new HashMap<>();
        for (String word : wordList) {
            for (int i = 0; i < word.length(); i++) {
                StringBuilder wordStringBuilder = new StringBuilder(word);
                wordStringBuilder.setCharAt(i, MATCHES_ANY_SINGLE_CHARACTER);
                String glob = wordStringBuilder.toString();
                List<String> list = globWordsMap.getOrDefault(glob, new ArrayList<>());
                list.add(word);
                globWordsMap.put(glob, list);
            }
        }
        Queue<String> wordQueue = new LinkedList<>();
        wordQueue.offer(beginWord);
        Set<String> visitedWords = new HashSet<>();
        visitedWords.add(beginWord);

        int step = 1;
        while (!wordQueue.isEmpty()) {
            step++;

            int wordQueueSizeSnapShot = wordQueue.size();
            for (int wordQueueIndex = 0; wordQueueIndex < wordQueueSizeSnapShot; wordQueueIndex++) {
                String word = wordQueue.poll();
                for (int i = 0; i < word.length(); i++) {
                    StringBuilder wordStringBuilder = new StringBuilder(word);
                    wordStringBuilder.setCharAt(i, MATCHES_ANY_SINGLE_CHARACTER);
                    String glob = wordStringBuilder.toString();
                    for (String adjacentWord : globWordsMap.get(glob)) {
                        if (adjacentWord.equals(endWord)) {
                            return step;
                        }
                        if (visitedWords.contains(adjacentWord)) {
                            continue;
                        }

                        wordQueue.offer(adjacentWord);
                        visitedWords.add(adjacentWord);
                    }
                }
            }
        }

        return 0;
    }
}
```

### 答えを見ずに解こうとした記録

なんとなくいける気がしたので、答えを見る前に2時間だけ頑張ってみました。残念ながらあるテストケースで TLE となりました。答えを見てから振り返ると、これじゃあいかんなという印象です。  
アルゴリズムだけでなく変数の命名等についても課題が多いままですが、一応最初の解答ということで記載しておきます。

```java
// 使った時間: 2時間
// 時間計算量: O(N^3)
// 空間計算量: O(N)
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        int ladderLength = 0;
        for (String word : wordList) {
            if (word.equals(endWord)) {
                ladderLength = getShortestTransformationSequence(beginWord, word, wordList);
            }
        }
        return ladderLength;
    }

    private int getShortestTransformationSequence(String beginWord, String endWord, List<String> wordList) {
        if (isTransformable(beginWord, endWord)) {
            return 2;
        }
        List<String> newWordList = createNewWordList(endWord, wordList);
        if (newWordList.isEmpty()) {
            return 0;
        }

        int shortestTransformationSequence = 0;
        for (String word : newWordList) {
            if (!isTransformable(word, endWord)) {
                continue;
            }
            int transformationSequence = getShortestTransformationSequence(beginWord, word, newWordList);
            if (transformationSequence > 0 && transformationSequence < shortestTransformationSequence || shortestTransformationSequence == 0) {
                shortestTransformationSequence = transformationSequence;
            }
        }

        if (shortestTransformationSequence == 0) {
            return 0;
        }
        return shortestTransformationSequence + 1;
    }

    private boolean isTransformable(String xWord, String yWord) {
        int difference = 0;
        char[] xChars = xWord.toCharArray();
        char[] yChars = yWord.toCharArray();
        for (int i = 0; i < xChars.length; i++) {
            if (xChars[i] != yChars[i]) {
                difference++;
            }
        }
        return difference <= 1;
    }

    private List<String> createNewWordList(String endWord, List<String> wordList) {
        String targetWord = "";
        for (String word : wordList) {
            if (word.equals(endWord)) {
                targetWord = word;
            }
        }
        if (targetWord.equals("")) {
            return new ArrayList<>();
        }
        List<String> transformableWordList = new ArrayList<>(wordList);
        transformableWordList.remove(targetWord);
        return transformableWordList;
    }
}
```

今振り返ると次のような感想が思い浮かびます:

- そもそも DFS アプローチで行こうとしているのが筋がわるい。最短経路探索なのでまず BFS が思い浮かぶべき。
    - [fhiyo さんの「エッジの重みがすべて1のグラフ上の最短距離を求めるのだからBFSがまず候補に入るはず。」という思考ログ](https://github.com/fhiyo/leetcode/pull/22/files)を見て、自分もこれがまず頭に浮かぶようになりたいなと思った
- キャッシュしないと厳しいとわかっていたのだから最初から実装に組み込むべきだった

実装当時は以下のようなことを考えて実装していました:

- endWord から beginWord の adjacent pair に到れるかどうかを確認し、到った中で最短 transformation sequence を返す実装にすれば良さそう
- キャッシュしてあげないと TLE になりそうだが、その実装は一旦処理が完成してからにした方が順序としては望ましいかな
- 可読性を考えると再帰関数にするのが良い気がする
    - 再帰の深さは wordList の最大要素数 5000 が上限となるはずなので、 Java なら恐らくスタックオーバーフローにはならないだろう
- 必要な関数は次の3つになるだろうか:
    - 再帰関数 getShortestTransformationSequence()
    - 2つの word が adjacent pair であるかどうかを判定する isTransformable()
    - 対象の word を除いたリストを作成する createNewWordList()
- いくつかケースが通るようになったがやはり要素数が多いと TLE になった
- ここで確保してた2時間が経ったので終了

## Step 2

### パターンと文字列の対応を用いた実装 (Step 1 の解法の修正)

```java
// 時間計算量: O(N^2)
// 空間計算量: O(N)
class Solution {
    private static final int NO_SEQUENCE_FOUND = 0;

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Map<String, List<String>> globWordsMap = new HashMap<>();
        // 引数に与えられた値を書き換えることを良しとしています
        wordList.add(beginWord);
        for (String word : wordList) {
            for (int i = 0; i < word.length(); i++) {
                String globPattern = createGlobPattern(i, word);
                List<String> words = globWordsMap.getOrDefault(globPattern, new ArrayList<>());
                words.add(word);
                globWordsMap.put(globPattern, words);
            }
        }

        Queue<String> wordQueue = new ArrayDeque<>();
        wordQueue.offer(beginWord);
        Set<String> visitedWords = new HashSet<>();
        visitedWords.add(beginWord);

        int distance = 1;
        while (!wordQueue.isEmpty()) {
            distance++;
            int wordQueueSizeSnapshot = wordQueue.size();
            for (int i = 0; i < wordQueueSizeSnapshot; i++) {
                String word = wordQueue.poll();
                for (int j = 0; j < word.length(); j++) {
                    String globPattern = createGlobPattern(j, word);
                    for (String adjacentWord : globWordsMap.get(globPattern)) {
                        if (adjacentWord.equals(endWord)) {
                            return distance;
                        }
                        if (visitedWords.contains(adjacentWord)) {
                            continue;
                        }
                        wordQueue.offer(adjacentWord);
                        visitedWords.add(adjacentWord);
                    }
                }
            }
        }

        return NO_SEQUENCE_FOUND;
    }

    private String createGlobPattern(int index, String word) {
        StringBuilder patternBuilder = new StringBuilder(word);
        patternBuilder.setCharAt(index, '?');
        return patternBuilder.toString();
    }
}
```

- Levenshtein distance, Edit distance, Hamming distance についての知識があれば `step` よりも `distance` の方が役割を理解しやすいなと思ったので変数名を修正 ([こちらのコメント](https://github.com/Ryotaro25/leetcode_first60/pull/22#issuecomment-2255580125)とその引用から知りました)
- クラスの外から渡される引数 wordList を書き換える副作用があっていいのかは議論の余地ありなので、一応その点コメント

### Deque に単語と編集距離をもたせる BFS アプローチ

[torus さんの解答](https://github.com/TORUS0818/leetcode/pull/22/files?short_path=d25ca00#diff-d25ca007ef80e5e40d07985c432c899c349a715552d7ec202509fbf470586658)を参考に実装してみた。

```java
// 時間計算量: O(N^2)
// 空間計算量: O(N)
class Solution {
    private record WordDistance(String word, int distance) {};
    private static final int NO_SEQUENCE_EXISTS = 0;

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Deque<WordDistance> wordDistances = new ArrayDeque<>();
        wordDistances.addLast(new WordDistance(beginWord, 1));
        while (!wordDistances.isEmpty()) {
            WordDistance wordDistance = wordDistances.removeFirst();
            if (wordDistance.word.equals(endWord)) {
                return wordDistance.distance;
            }
            Set<String> addedWords = new HashSet<>();
            for (String targetWord : wordList) {
                if (!isTransformable(targetWord, wordDistance.word)) {
                    continue;
                }
                addedWords.add(targetWord);
                wordDistances.addLast(new WordDistance(targetWord, wordDistance.distance + 1));
            }
            wordList.removeAll(addedWords);
        }
        return NO_SEQUENCE_EXISTS;
    }

    private boolean isTransformable(String xWord, String yWord) {
        char[] xChars = xWord.toCharArray();
        char[] yChars = yWord.toCharArray();
        int difference = 0;
        // 両方の引数の文字数が等しいことを前提とした実装になります
        for (int i = 0; i < xChars.length; i++) {
            if (xChars[i] != yChars[i]) {
                difference++;
            }
            if (difference > 1) {
                return false;
            }
        }
        return true;
    }
}
```

- 自分も自然に思いつきそうだなという印象を持った
- パターンと文字列の対応を用いた実装と比べ実行時間は1桁多かった

## Step 3

パターンと文字列の対応を用いた実装にしました。

```java
// 解いた時間: 20分ぐらい
// 時間計算量: O(N^2)
// 空間計算量: O(N)
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        wordList.add(beginWord);
        Map<String, List<String>> adjacencyMap = new HashMap<>();
        for (String word : wordList) {
            for (int i = 0; i < word.length(); i++) {
                String globPattern = createGlobPattern(i, word);
                List<String> list = adjacencyMap.getOrDefault(globPattern, new ArrayList<>());
                list.add(word);
                adjacencyMap.put(globPattern, list);
            }
        }

        Deque<String> wordQueue = new ArrayDeque<>();
        wordQueue.addLast(beginWord);
        Set<String> visitedWords = new HashSet<>();
        visitedWords.add(beginWord);

        int distance = 1;
        while(!wordQueue.isEmpty()) {
            distance++;
            int wordQueueSizeSnapshot = wordQueue.size();
            for (int wordQueueIndex = 0; wordQueueIndex < wordQueueSizeSnapshot; wordQueueIndex++) {
                String word = wordQueue.removeFirst();
                for (int i = 0; i < word.length(); i++) {
                    String globPattern = createGlobPattern(i, word);
                    for (String adjacentWord : adjacencyMap.get(globPattern)) {
                        if (adjacentWord.equals(endWord)) {
                            return distance;
                        }
                        if (visitedWords.contains(adjacentWord)) {
                            continue;
                        }
                        wordQueue.addLast(adjacentWord);
                        visitedWords.add(adjacentWord);
                    }
                }
            }
        }

        return 0;
    }

    private String createGlobPattern(int index, String word) {
        StringBuilder globPatternBuilder = new StringBuilder(word);
        globPatternBuilder.setCharAt(index, '?');
        return globPatternBuilder.toString();
    }
}
```

- 書く量が多く大変だったからか、 0 を定数化したりコメントを書くのが抜けていた

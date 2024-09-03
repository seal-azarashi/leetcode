# 127. Word Ladder

LeetCode URL: https://leetcode.com/problems/word-ladder/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

### 答えを見て解いたときの実装

```java
// 時間計算量: O(n^2)
// 空間計算量: O(n)
class Solution {
    private static final char MATCHES_SINGLE_CHARACTER = '?';

    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        wordList.add(beginWord);
        Map<String, List<String>> globWordsMap = new HashMap<>();
        for (String word : wordList) {
            for (int i = 0; i < word.length(); i++) {
                StringBuilder wordStringBuilder = new StringBuilder(word);
                wordStringBuilder.setCharAt(i, MATCHES_SINGLE_CHARACTER);
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
                    wordStringBuilder.setCharAt(i, MATCHES_SINGLE_CHARACTER);
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

なんとなくいける気がしたので、答えを見る前に2時間だけ頑張ってみました。残念ながらあるテストケースで TLE となりました。  
アルゴリズムだけでなく変数の命名等についても課題が多いままですが、一応最初の解答ということで記載しておきます。

```java
// 使った時間: 2時間
// 時間計算量: O(n^2)
// 空間計算量: O(n)
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

以下のようなことを考えて実装していました:

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

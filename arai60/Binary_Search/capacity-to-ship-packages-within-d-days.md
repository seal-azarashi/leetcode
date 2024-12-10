# 33. Search in Rotated Sorted Array

LeetCode URL: https://leetcode.com/problems/search-in-rotated-sorted-array/description/

この問題は Java で解いています。  
各解法において、メソッドが属するクラスとして `Solution` を定義していますが、これは Java の言語仕様に従い、コードを実行可能にするために必要なものです。このクラス自体には特定の意味はなく、単にメソッドを組織化し、実行可能にするためのものです。

## Step 1

次のように考えながら解きました

- 最小の capacity を算出するのに何を考えればいいだろうか
- まずある程度あたりを付けたいので、考えられる最小値と最大値の候補を出す
    - 最小値: 最大の重さの package と同じ重さ
        - これを下回るとそもそも運べなくなる
    - 最大値: 全 package の総重量
        - 一日で運び切るのに必要な最小値とも言える
        - もっと絞り込むことも出来そうだが、処理全体のオーダーには影響しないので一旦このまま進める
- これらの間のどこかに答えがある
- これ以上範囲を絞り込む要素はなさそうなので、二分探索で検索をする

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int minCapacity = findMaxPackageWeight(weights);
        int maxCapacity = calculateTotalPackageWeight(weights);
        while (minCapacity < maxCapacity) {
            int middleCapacity = minCapacity + (maxCapacity - minCapacity) / 2;
            if (isAbleToShip(weights, days, middleCapacity)) {
                maxCapacity = middleCapacity;
            } else {
                minCapacity = middleCapacity + 1;
            }
        }
        return minCapacity;
    }

    private int findMaxPackageWeight(int[] weights) {
        int maxPackageWeight = weights[0];
        for (int weight : weights) {
            maxPackageWeight = Math.max(maxPackageWeight, weight);
        }
        return maxPackageWeight;
    }

    private int calculateTotalPackageWeight(int[] weights) {
        int totalPackageWeight = 0;
        for (int weight : weights) {
            totalPackageWeight += weight;
        }
        return totalPackageWeight;
    }

    private boolean isAbleToShip(int[] weights, int days, int capacity) {
        int currentTotalAmount = 0;
        int daysSpent = 0;
        for (int weight : weights) {
            currentTotalAmount += weight;
            if (currentTotalAmount > capacity) {
                daysSpent++;
                currentTotalAmount = weight;
            }
        }
        daysSpent++;
        return daysSpent <= days;
    }
}
```

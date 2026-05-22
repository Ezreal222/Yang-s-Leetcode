# 0135. Candy / 分发糖果

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Greedy, Array · 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/candy/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given children's ratings, assign candies under: each child has ≥ 1 candy, and any child with a higher rating than an adjacent child gets more candies than that neighbor. Return the minimum total candies.

**中文**: 给一组孩子的 rating, 分糖要求: 每个孩子至少 1 颗; 比相邻 (左/右) rating 高的孩子, 糖比那个邻居多. 求最少总糖数.

## Key Insights

1. **两遍扫拆解左右约束 / Two passes split the constraint**

    单次扫只能照顾一边: 从左往右扫保证"右 > 左 时右糖更多", 但管不了"左 > 右 时左糖更多". 拆成两遍:

    - **第一遍 (左→右)**: `if (ratings[i] > ratings[i-1]) candies[i] = candies[i-1] + 1`. 满足所有"右边比左边高" 的约束.
    - **第二遍 (右→左)**: `if (ratings[i] > ratings[i+1]) candies[i] = max(candies[i], candies[i+1] + 1)`. 满足所有"左边比右边高" 的约束.

    两遍合起来, 每个相邻关系都被覆盖至少一遍.

2. **`max` 不可少: 保住第一遍的成果 / Why `max` is critical**

    Yang flagged 的关键: 第二遍**不能直接** `candies[i] = candies[i+1] + 1`, 否则会**破坏**第一遍已经建立的"左边约束". 反例 `[1, 3, 4, 2]`:

    - 第一遍: `[1, 2, 3, 1]`.
    - 第二遍 i=2: ratings[2]=4 > ratings[3]=2. 若不用 max, `candies[2] = 1 + 1 = 2` — 但此时 candies[1]=2, 违反 ratings[2]=4 > ratings[1]=3 应有的 candies[2] > candies[1].
    - 用 `max(candies[i], candies[i+1] + 1)` 取 max(3, 2) = 3, 保住了左边约束.

    口诀: **第二遍只能加码, 不能覆盖**.

3. **跟 0053 / 0134 那种"一遍扫" 贪心的区别 / Two-pass is its own pattern**

    简单贪心一遍就够 (Kadane, gas station). 这题约束是**双向**的, 必须双向扫. 同款思路应用: 0042 Trapping Rain Water (左右两遍扫求 max), 0238 Product of Array Except Self (左右两遍扫求前缀后缀积).

    模板: **每遍扫只解决一个方向的约束, 用 max/min 让两遍结果兼容**.

4. **复杂度 O(n) 时间 + O(n) 空间 / Linear in both**

    两次线性扫 + 一次累加 = O(n). candies 数组 O(n). 没有 in-place 优化 (除非愿意写两个变量交替), 但实际不必.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int candy(vector<int>& ratings) {
            int n = ratings.size();
            vector<int> candies(n, 1);                              // 每人至少 1 颗
            for (int i = 1; i < n; i++) {
                if (ratings[i] > ratings[i - 1])
                    candies[i] = candies[i - 1] + 1;                // 左→右: 右高就 +1
            }
            for (int i = n - 2; i >= 0; i--) {
                if (ratings[i] > ratings[i + 1])
                    candies[i] = max(candies[i], candies[i + 1] + 1); // 右→左: 必须 max
            }
            return accumulate(candies.begin(), candies.end(), 0);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def candy(self, ratings: list[int]) -> int:
            n = len(ratings)
            candies = [1] * n                                       # 每人 1 颗起步
            for i in range(1, n):
                if ratings[i] > ratings[i - 1]:
                    candies[i] = candies[i - 1] + 1
            for i in range(n - 2, -1, -1):                          # range(start, stop, step=-1) 倒序
                if ratings[i] > ratings[i + 1]:
                    candies[i] = max(candies[i], candies[i + 1] + 1)
            return sum(candies)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} ratings
     * @return {number}
     */
    var candy = function(ratings) {
        const n = ratings.length;
        const candies = new Array(n).fill(1);
        for (let i = 1; i < n; i++) {
            if (ratings[i] > ratings[i - 1]) candies[i] = candies[i - 1] + 1;
        }
        for (let i = n - 2; i >= 0; i--) {
            if (ratings[i] > ratings[i + 1]) {
                candies[i] = Math.max(candies[i], candies[i + 1] + 1);
            }
        }
        return candies.reduce((a, b) => a + b, 0);
    };
    ```

## Complexity

- **Time**: O(n) — 两遍扫 + 累加.
- **Space**: O(n) — candies 数组.

## 相关题目

- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) / [0134. Gas Station](../0134-gas-station/README.md) — 一遍扫贪心对照
- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门
- 0042\. Trapping Rain Water (待补) — 同款"左右两遍扫求 max" 模板
- 0238\. Product of Array Except Self (待补) — 同款两遍扫 (前缀积 + 后缀积)
- 0581\. Shortest Unsorted Continuous Subarray (待补) — 同款"双向扫各管一个方向"

# 0474. Ones and Zeroes / 一和零

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, 2D 0/1 Knapsack · 动态规划, 二维 01 背包
    - **Link**: [LeetCode](https://leetcode.com/problems/ones-and-zeroes/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给二进制串数组 `strs`, 整数 `m` (0 的预算) 和 `n` (1 的预算). 选出最多的字符串, 使所选字符串里 `0` 的总数 ≤ `m`, `1` 的总数 ≤ `n`. 返回**最多能选几个**.

**中文**: 二进制串数组 `strs`, 选最多字符串使 `0` 总数 ≤ `m`, `1` 总数 ≤ `n`, 返回最多选几个.

## Key Insights

1. **二维 0/1 背包 / Two-dimensional 0/1 knapsack**

    每个字符串是一件物品, 但有**两个重量维度**: `(zeros, ones)`. 价值都是 1 (选 1 件加 1). 容量也变两维 `(m, n)`.

    > 一维 0/1 背包 ([0416](../0416-partition-equal-subset-sum/README.md)/[1049](../1049-last-stone-weight-ii/README.md)/[0494](../0494-target-sum/README.md)) 的直接升级: **状态多一个轴, 转移多一个减**.

2. **状态 / 转移 / Definition & recurrence**

    $$dp[i][j] = \text{最多能选几个 (用 ≤ i 个 0, ≤ j 个 1)}$$

    转移 — 当前串选或不选:

    $$dp[i][j] = \max(\underbrace{dp[i][j]}_{\text{不选}},\ \underbrace{dp[i - \text{zeros}][j - \text{ones}] + 1}_{\text{选}})$$

    > 跟一维背包的 `dp[j] = max(dp[j], dp[j-w] + v)` 一一对应, 只是把 `j` 拆成 `(i, j)` 两轴, `w` 拆成 `(zeros, ones)`.

3. **🔑 两个容量轴都要倒序 / Iterate BOTH capacities backwards**

    跟一维一样, 物品只能用一次 → 两个 for 都从大到小. 任一正序都会让当前串被重复"装入".

    ```cpp
    for (int i = m; i >= zeros; i--)
        for (int j = n; j >= ones; j--)
            ...
    ```

    > 0/1 背包倒序规则在多维下不变 — **每个容量轴各自倒序**.

4. **value = 1, weight = (zeros, ones) — 注意维度归属 / Don't confuse weight with value**

    "选最多字符串" 的目标里, 每件物品价值都是 `+1`, 不是 `+ones` 之类. 跟 [0416](../0416-partition-equal-subset-sum/README.md) "重量=价值" 的特例不一样.

    > 写背包前永远先问: **"物品的 weight 是什么? value 是什么? 容量是什么?"** 三件套不混就不会错.

5. **预处理 zeros/ones 逐串扫一次 / Preprocess char counts per string**

    每个串 O(len) 数一遍 `0/1` 个数. 总预处理 O(总字符数), 远小于 DP 主循环 O(strs.size × m × n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findMaxForm(vector<string>& strs, int m, int n) {
            vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
            for (const string& s : strs) {
                int zeros = 0, ones = 0;
                for (char c : s) {
                    c == '1' ? ones++ : zeros++;
                }
                // 二维都倒序: 0/1 背包规则
                for (int i = m; i >= zeros; i--) {
                    for (int j = n; j >= ones; j--) {
                        dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
                    }
                }
            }
            return dp[m][n];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findMaxForm(self, strs: list[str], m: int, n: int) -> int:
            # [[0]*(n+1) for _ in range(m+1)]: 二维 0 矩阵, m+1 行 n+1 列
            # 别写 [[0]*(n+1)] * (m+1) — 那是 m+1 行指向同一行, 改一行全变
            dp = [[0] * (n + 1) for _ in range(m + 1)]
            for s in strs:
                # s.count('0') / s.count('1') 内置 O(len), 比手写 for 简洁
                zeros, ones = s.count('0'), s.count('1')
                # 二维倒序: range(m, zeros-1, -1) = for(i=m; i>=zeros; i--)
                for i in range(m, zeros - 1, -1):
                    for j in range(n, ones - 1, -1):
                        dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1)
            return dp[m][n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} strs
     * @param {number} m
     * @param {number} n
     * @return {number}
     */
    var findMaxForm = function(strs, m, n) {
        // Array.from({length: m+1}, () => new Array(n+1).fill(0)) 创建独立每行
        // 别写 Array(m+1).fill(Array(n+1).fill(0)) — 所有行指向同一对象!
        const dp = Array.from({length: m + 1}, () => new Array(n + 1).fill(0));
        for (const s of strs) {
            let zeros = 0, ones = 0;
            for (const c of s) c === '1' ? ones++ : zeros++;
            for (let i = m; i >= zeros; i--) {
                for (let j = n; j >= ones; j--) {
                    dp[i][j] = Math.max(dp[i][j], dp[i - zeros][j - ones] + 1);
                }
            }
        }
        return dp[m][n];
    };
    ```

## Complexity

- **Time**: O(K × m × n + sum_len), K = `strs.size()`. 主循环 K × m × n 主导.
- **Space**: O(m × n) — 二维滚动表 (滚掉了"考虑前几件物品" 那一维).

## 相关题目

- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 一维 0/1 背包母题
- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 同 0/1 背包, 求最小差
- [0494. Target Sum](../0494-target-sum/README.md) — 0/1 背包计数变体
- 0879\. Profitable Schemes (待补) — 二维 0/1 背包 + 计数, 本题的"双胞胎升级"
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包对照 (正序 vs 本题倒序)

# 1547. Minimum Cost to Cut a Stick / 切棍子的最小成本

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval, Sorting · 动态规划, 区间, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 一根长为 `n` 的木棍 (位置 0 到 n). 给数组 `cuts` 表示要切的位置. 每次切的**代价 = 当前被切的那段木棍的长度**. 切的顺序自定, 求**最小总代价**.

**中文**: 棍长 n, `cuts` 是要切的位置. 每次切代价 = 当前切的那段长度. 求最小总代价.

## Key Insights

1. **🔑 跟 [0312 戳气球](../0312-burst-balloons/README.md) / [1039 三角剖分](../1039-minimum-score-triangulation-of-polygon/README.md) 同模板 / Same template as 0312/1039**

    三层循环, 枚举区间内某点切两段, 子问题独立 + 含该点的贡献. **本题特点**:

    - 木棍两端 `0` 和 `n` 必须加入 `cuts` 当**哨兵**, 让区间端点表达"当前段的左右边界".
    - **`cuts` 排序**: 让"在 cuts[i] 到 cuts[j] 之间所有切点" 这个集合恰好是 cuts 数组上索引 `(i, j)` 区间 — 区间 DP 才能自然成立.

    > **哨兵 + 排序** 是这题跟 0312 (一端补 1) / 1039 (顶点本身就排序) 的关键差别. 没排序则区间没意义.

2. **状态: `dp[i][j] = 把 cuts[i] 到 cuts[j] 之间所有切点切完的最小代价` / Min cost between two boundaries**

    `cuts[i]` 和 `cuts[j]` 是当前段的左右端点 (已经被切出来或本来就是边界 0/n). 之间所有切点都要切完. 答案是 `dp[0][m-1]` (m = 加哨兵后的 cuts 长度).

3. **🔑 转移: 枚举第一个 (或最后一个) 切的位置 k / Enumerate first/last cut**

    在区间 `(i, j)` 内, 选一个 `k` (`i < k < j`) **第一个**切. 第一刀代价 = 当前段长 = `cuts[j] - cuts[i]`. 切完分成两段 (i, k) 和 (k, j), 各自递归.

    $$dp[i][j] = \min_{i < k < j} \big( dp[i][k] + dp[k][j] + (cuts[j] - cuts[i]) \big)$$

    > **"第一个" 还是"最后一个" 都行** — 因为代价只跟"当时这段的长度" 有关, 当 (i, j) 这段被独立处理时, 切第一刀代价就是 cuts[j] - cuts[i]. 跟 [0312 枚举最后破](../0312-burst-balloons/README.md) 是同一思想.

4. **边界: `j - i < 2` (区间内无切点) → dp = 0 / Empty interior**

    没切点要切就不花钱. 默认值 0, 跟 `for len = 2; len < m;` 的循环边界自洽.

5. **按区间长度遍历 / Iterate by interval length**

    跟 [1000 合并石头](../1000-minimum-cost-to-merge-stones/README.md) 一样用 `for len = 2; len < m; len++` — 因为转移需要更短区间先填好. 等价的"i 倒序 j 顺序" 也行, 看个人习惯.

6. **`(cuts[j] - cuts[i])` 是常数, 跟 k 无关 / Constant cost per (i, j)**

    内层循环里这一项不依赖 k. 可以提到 k 循环外算一次, 微小优化:

    ```cpp
    int cost = cuts[j] - cuts[i];
    for (int k = i + 1; k < j; k++) {
        dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cost);
    }
    ```

    > 这跟 [0312](../0312-burst-balloons/README.md) 的 `balloons[i] * balloons[k] * balloons[j]` 不一样 — 那个**依赖 k**, 不能提出.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minCost(int n, vector<int>& cuts) {
            // 两端补哨兵 + 排序 — 让 cuts 索引区间对应木棍上的段
            cuts.push_back(0);
            cuts.push_back(n);
            sort(cuts.begin(), cuts.end());
            int m = cuts.size();

            vector<vector<int>> dp(m, vector<int>(m, 0));

            for (int len = 2; len < m; len++) {                    // 按区间长度
                for (int i = 0; i + len < m; i++) {
                    int j = i + len;
                    dp[i][j] = INT_MAX;                            // 求 min 先置哨兵
                    int cost = cuts[j] - cuts[i];                  // 当前段长, k 无关, 提出来
                    for (int k = i + 1; k < j; k++) {
                        dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j] + cost);
                    }
                }
            }
            return dp[0][m - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minCost(self, n: int, cuts: list[int]) -> int:
            # 两端补哨兵 + 排序
            cuts = sorted(cuts + [0, n])
            m = len(cuts)
            dp = [[0] * m for _ in range(m)]

            for length in range(2, m):
                for i in range(m - length):
                    j = i + length
                    cost = cuts[j] - cuts[i]                       # 当前段长 (k 无关)
                    # min(...) + 生成器一行算所有 k 的最小; 等价 C++ 内层 for
                    dp[i][j] = min(dp[i][k] + dp[k][j] for k in range(i + 1, j)) + cost
            return dp[0][m - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @param {number[]} cuts
     * @return {number}
     */
    var minCost = function(n, cuts) {
        cuts.push(0, n);                                            // spread 不能 ., push 多参
        cuts.sort((a, b) => a - b);                                 // ⚠ 数字必须传 compareFn
        const m = cuts.length;
        const dp = Array.from({length: m}, () => new Array(m).fill(0));

        for (let len = 2; len < m; len++) {
            for (let i = 0; i + len < m; i++) {
                const j = i + len;
                const cost = cuts[j] - cuts[i];
                let best = Infinity;
                for (let k = i + 1; k < j; k++) {
                    best = Math.min(best, dp[i][k] + dp[k][j]);
                }
                dp[i][j] = best + cost;
            }
        }
        return dp[0][m - 1];
    };
    ```

## Complexity

- **Time**: O(m³) where m = `cuts.size() + 2`.
- **Space**: O(m²).

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 结构最相似, 枚举"最后破" + 哨兵
- [1039. Minimum Score Triangulation of Polygon](../1039-minimum-score-triangulation-of-polygon/README.md) — 同款"枚举中间点切两段"
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 区间 DP 进阶, 3D 状态
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) / [0516](../0516-longest-palindromic-subsequence/README.md) — 区间 DP 入门
- 0664\. Strange Printer (待补) — 区间 DP, 同款"分段处理"
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 区间 DP, min-max 博弈

# 1130. Minimum Cost Tree From Leaf Values / 叶值的最小代价生成树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval, Monotonic Stack · 动态规划, 区间, 单调栈
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给数组 `arr`, 元素**按顺序** 作为某二叉树的叶子. 每个内部节点的值 = 左子树最大叶 × 右子树最大叶, 树的代价 = 所有内部节点之和. 求**最小代价**.

**中文**: `arr` 按顺序做二叉树叶子, 内部节点 = 左右子树最大叶相乘, 求所有内部节点和的最小值.

## Key Insights

1. **🔑 区间 DP + 枚举切点 / Interval DP, enumerate split**

    叶子顺序固定 → 树的形状唯一由"每次切哪刀" 决定. 跟 [1547 切棍](../1547-minimum-cost-to-cut-a-stick/README.md) / [0312 戳气球](../0312-burst-balloons/README.md) 同结构: 枚举 `k` 把区间切成左 `[i, k]` 和右 `[k+1, j]`, 各自递归.

    > 看到"按顺序构造二叉树 / 表达式 / 分组" → 立刻反应"枚举切点" 区间 DP.

2. **状态: `dp[i][j] = arr[i..j] 作为叶子构建子树的最小内部节点和` / Min cost to build subtree on leaves [i, j]**

    单叶 `dp[i][i] = 0` (没有内部节点). 答案是 `dp[0][n-1]`.

3. **🔑 转移: 分裂代价 = 左 max × 右 max + 两子树各自代价 / Split cost**

    切在 `k` 后, 当前根节点的值 = `max(arr[i..k]) × max(arr[k+1..j])`, 加上两子树各自 DP:

    $$dp[i][j] = \min_{i \le k < j} \big( dp[i][k] + dp[k+1][j] + \max(arr[i..k]) \times \max(arr[k+1..j]) \big)$$

4. **🔑 预处理 max 表 O(n²) → 转移 O(1) 查 max / Precompute max table**

    内层转移要算 `max(arr[i..k])` 和 `max(arr[k+1..j])`. 朴素每次 O(n) 求 max, 总 O(n⁴). 预处理 `mx[i][j] = max(arr[i..j])` 表 (O(n²) 时间空间), 转移降到 O(1) 查, 总 O(n³).

    Yang 用递推 `mx[i][j] = max(mx[i][j-1], arr[j])` — 一遍 O(n²) 填表.

    > **"区间 max" 类预处理** 是区间 DP 优化的标准操作, 跟 [1547 cuts[j]-cuts[i] 提循环外](../1547-minimum-cost-to-cut-a-stick/README.md) / [1000 前缀和](../1000-minimum-cost-to-merge-stones/README.md) 同精神 — **常数 / 区间统计先算好**.

5. **按区间长度遍历 / Iterate by interval length**

    跟所有区间 DP 一样, `for len = 2; len <= n; len++` — 短区间先填好.

6. **替代解法: O(n) 单调栈贪心 / Greedy monotonic stack alternative**

    本题有 O(n) 单调栈解: 维护一个**单调递减栈**, 遇到比栈顶大的 `arr[i]` 时, 栈顶最小叶必须被"配对", 选其左右较小者相乘加入答案. 思想是"小叶子尽早被吃掉, 跟它最小的邻居配".

    DP 是通解, 单调栈是该特定结构的 hack. 本题作为区间 DP 模板学习, **DP 解更适合家族对照**. 学过区间 DP 后再去理解单调栈解会更顺.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int mctFromLeafValues(vector<int>& arr) {
            int n = arr.size();
            // 预处理: mx[i][j] = max(arr[i..j]), 递推 O(n²)
            vector<vector<int>> mx(n, vector<int>(n, 0));
            for (int i = 0; i < n; i++) {
                mx[i][i] = arr[i];
                for (int j = i + 1; j < n; j++) {
                    mx[i][j] = max(mx[i][j - 1], arr[j]);
                }
            }

            vector<vector<int>> dp(n, vector<int>(n, 0));         // 单叶 dp = 0
            for (int len = 2; len <= n; len++) {                   // 按长度
                for (int i = 0; i + len - 1 < n; i++) {
                    int j = i + len - 1;
                    dp[i][j] = INT_MAX;                            // min 先置 INF
                    for (int k = i; k < j; k++) {                  // 枚举切点
                        int cost = dp[i][k] + dp[k + 1][j] + mx[i][k] * mx[k + 1][j];
                        dp[i][j] = min(dp[i][j], cost);
                    }
                }
            }
            return dp[0][n - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def mctFromLeafValues(self, arr: list[int]) -> int:
            n = len(arr)
            # 预处理 max[i][j] 表
            mx = [[0] * n for _ in range(n)]
            for i in range(n):
                mx[i][i] = arr[i]
                for j in range(i + 1, n):
                    mx[i][j] = max(mx[i][j - 1], arr[j])

            dp = [[0] * n for _ in range(n)]
            for length in range(2, n + 1):
                for i in range(n - length + 1):
                    j = i + length - 1
                    # min(...) + 生成器一行算所有 k 的最小; 等价 C++ 内层 for
                    dp[i][j] = min(
                        dp[i][k] + dp[k + 1][j] + mx[i][k] * mx[k + 1][j]
                        for k in range(i, j)
                    )
            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} arr
     * @return {number}
     */
    var mctFromLeafValues = function(arr) {
        const n = arr.length;
        const mx = Array.from({length: n}, () => new Array(n).fill(0));
        for (let i = 0; i < n; i++) {
            mx[i][i] = arr[i];
            for (let j = i + 1; j < n; j++) {
                mx[i][j] = Math.max(mx[i][j - 1], arr[j]);
            }
        }

        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        for (let len = 2; len <= n; len++) {
            for (let i = 0; i + len - 1 < n; i++) {
                const j = i + len - 1;
                let best = Infinity;
                for (let k = i; k < j; k++) {
                    best = Math.min(best, dp[i][k] + dp[k + 1][j] + mx[i][k] * mx[k + 1][j]);
                }
                dp[i][j] = best;
            }
        }
        return dp[0][n - 1];
    };
    ```

## Complexity

- **Time**: O(n³) (DP) — `n²` 状态 × O(1) 转移 (查 max), 加 O(n²) 预处理. 单调栈解可 O(n).
- **Space**: O(n²) (mx + dp 两个表).

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP, 枚举"最后破"
- [1039. Minimum Score Triangulation of Polygon](../1039-minimum-score-triangulation-of-polygon/README.md) — 同款枚举切点 + 三点乘积
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 同款枚举切点 + 段长
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 区间 DP, 3D 进阶
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 区间 DP, min-max 博弈
- 0496\. Next Greater Element I (待补) — 单调栈基础, 跟本题 O(n) 解相关
- 0084\. Largest Rectangle in Histogram (待补) — 单调栈经典

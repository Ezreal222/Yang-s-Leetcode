# 0062. Unique Paths / 不同路径

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Math, Combinatorics · 动态规划, 数学, 组合
    - **Link**: [LeetCode](https://leetcode.com/problems/unique-paths/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `m × n` 网格, 机器人从左上 `(0,0)` 走到右下 `(m-1, n-1)`, 每步只能**向右**或**向下**. 求不同路径数.

**中文**: `m × n` 网格, 从左上到右下, 每步只能向右或向下, 求路径条数.

## Key Insights

1. **从二维 DP 入门: 两个方向决定 dp 表 / Step into 2D DP**

    一维 DP 是"一条直线上滚动", 二维 DP 是"网格上滚动". 状态:

    $$dp[i][j] = \text{到达 } (i, j) \text{ 的路径数}$$

    转移只有两个来源 — 从上来或从左来:

    $$dp[i][j] = dp[i-1][j] + dp[i][j-1]$$

    > 模板: 网格 DP 的转移基本只看"邻居", 状态怎么定看题目走法.

2. **初始化: 第一行第一列全是 1 / Edge cells: only one way**

    第一行只能一路向右, 第一列只能一路向下, 路径都唯一. 所以 `dp[i][0] = dp[0][j] = 1`. 这是网格 DP 的标配, 别漏.

    > 也可以加哨兵 `dp[0][0] = 1` + 转移从 (0,0) 开始, 但那要单独处理 `i-1<0` / `j-1<0`. 直接初始化第一行列更干净.

3. **遍历顺序: 从左上到右下 / Top-left to bottom-right**

    `dp[i][j]` 依赖 `dp[i-1][j]` (上) 和 `dp[i][j-1]` (左) — 都在左上方. **行优先 + 列优先**任一顺序都满足"先算依赖". 反过来就错.

4. **空间压缩到 O(n) / Rolling 1D array**

    `dp[i][j]` 只用到本行左边 + 上一行同列, 一维数组够:

    ```cpp
    dp[j] = dp[j] + dp[j-1];   // 右边 dp[j] 是"上一行" (覆盖前), dp[j-1] 是"本行左边"
    ```

    更新顺序必须**从左到右**, 这样 `dp[j-1]` 已经是新值 (本行左), `dp[j]` 还是旧值 (上一行同列).

    > 滚动数组的核心: **看每个位置写入前能不能算清"它依赖的值还存不存在"**. 写错顺序 = 用了未来值.

5. **组合数 O(m+n) 直接解 / Combinatorial closed form**

    要走 `m-1` 次"下" + `n-1` 次"右", 共 `m+n-2` 步, 选哪几步"下" 就定了:

    $$\binom{m+n-2}{m-1}$$

    实现要防溢出 — 边乘边除. 留作进阶, 对 LeetCode 范围 (m,n ≤ 100) DP 完全够.

## Solution

=== "C++"
    === "v1: 二维 DP (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int uniquePaths(int m, int n) {
                vector<vector<int>> dp(m, vector<int>(n, 0));
                for (int i = 0; i < m; i++) dp[i][0] = 1;          // 第一列: 只能一路向下
                for (int j = 0; j < n; j++) dp[0][j] = 1;          // 第一行: 只能一路向右
                for (int i = 1; i < m; i++) {
                    for (int j = 1; j < n; j++) {
                        dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
                    }
                }
                return dp[m - 1][n - 1];
            }
        };
        ```

    === "v2: 滚动一维 O(n)"
        ```cpp
        class Solution {
        public:
            int uniquePaths(int m, int n) {
                vector<int> dp(n, 1);                              // 初始为第一行: 全 1
                for (int i = 1; i < m; i++) {
                    for (int j = 1; j < n; j++) {
                        dp[j] += dp[j - 1];                        // dp[j]=上行同列(旧), dp[j-1]=本行左(新)
                    }
                }
                return dp[n - 1];
            }
        };
        ```

    === "v3: 组合数 O(m+n)"
        ```cpp
        class Solution {
        public:
            int uniquePaths(int m, int n) {
                // C(m+n-2, m-1) — 边乘边除防溢出
                long long ans = 1;
                int down = m - 1, total = m + n - 2;
                for (int i = 1; i <= down; i++) {
                    ans = ans * (total - down + i) / i;            // 分子 (n..n+down-1), 分母 (1..down)
                }
                return (int)ans;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def uniquePaths(self, m: int, n: int) -> int:
            # 一维滚动数组. dp[j] 在循环里既代表"上一行的 dp[i-1][j]" (访问时), 又代表"本行的 dp[i][j]" (写回时)
            # 初值全 1 = 第一行 (只能一路向右)
            dp = [1] * n
            for i in range(1, m):
                for j in range(1, n):
                    dp[j] += dp[j - 1]                             # dp[j] 旧值 + 本行左
            return dp[-1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} m
     * @param {number} n
     * @return {number}
     */
    var uniquePaths = function(m, n) {
        // Array(n).fill(1): 长度 n, 全填 1. 等价 C++ vector<int>(n, 1)
        // 不能写 Array(n) — 那是稀疏数组, 后续 dp[j-1] = undefined + ...
        const dp = new Array(n).fill(1);
        for (let i = 1; i < m; i++) {
            for (let j = 1; j < n; j++) {
                dp[j] += dp[j - 1];
            }
        }
        return dp[n - 1];
    };
    ```

## Complexity

- **Time**: O(m × n) (DP) / O(min(m, n)) (组合).
- **Space**: O(m × n) (v1) / O(n) (v2) / O(1) (v3).

## 相关题目

- [0063. Unique Paths II](../0063-unique-paths-ii/README.md) — 加障碍, 转移多一句"`obstacleGrid[i][j]==1` 则 `dp[i][j]=0`"
- 0064\. Minimum Path Sum (待补) — 同网格 DP, `+` 换成 `min` + `grid[i][j]`
- 0120\. Triangle (待补) — 倒三角网格, 同思路
- 0174\. Dungeon Game (待补) — 逆向 DP, 从右下倒推

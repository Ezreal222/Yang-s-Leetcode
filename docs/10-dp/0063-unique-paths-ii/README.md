# 0063. Unique Paths II / 不同路径 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Array · 动态规划, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/unique-paths-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 同 [0062](../0062-unique-paths/README.md), 但网格中 `1` 是**障碍**, 不能踩. 求左上到右下的路径数.

**中文**: 在 [0062 不同路径](../0062-unique-paths/README.md) 基础上, 网格 `1` 是障碍, 求路径条数.

## Key Insights

1. **状态/转移基本不变, 只在障碍格清零 / Same DP, set obstacle cells to 0**

    $$dp[i][j] = \begin{cases} 0, & \text{grid}[i][j] = 1 \\ dp[i-1][j] + dp[i][j-1], & \text{otherwise} \end{cases}$$

    障碍格不可达 → `dp = 0`. 转移本身没变, 只是来自上方/左方的某条路可能被截断 (那条路对应的 dp 已经是 0).

    > 加约束的 DP 通常**不动转移结构**, 只在 `dp` 赋值前判断"这格能不能要".

2. **初始化关键: 第一行/列遇障碍后**全部**置 0 / Edge init breaks at first obstacle**

    第一行只能向右, 一旦中间有障碍, **后面所有格都到不了** → 都是 0. 列同理. 写法是 `for` + `break` (Yang 的写法), 等价于"在障碍前全 1, 之后全 0".

    > 常见错: 只把障碍那格设 0, 后面继续设 1 — 错了, 因为前面被堵死, 后面也到不了. **边界初始化必须考虑可达性**.

3. **起点/终点本身就是障碍 → 0 / Guard the corners**

    若 `grid[0][0] == 1` 或 `grid[m-1][n-1] == 1`, 答案就是 0. Yang 的 `for ... break` 自然处理了起点 — 第一格判障碍则 `dp[0][0] = 0`, 后面初始化全跳过. 终点的话, 因为前面所有转移都把它对应的 0 传上来, 也自然得 0. 不用单独 if.

4. **空间压缩到 O(n) / Rolling 1D**

    跟 [0062 v2](../0062-unique-paths/README.md) 同套路, 加一句"障碍清零":

    ```cpp
    if (obstacleGrid[i][j] == 1) dp[j] = 0;
    else if (j > 0)              dp[j] += dp[j - 1];
    ```

    第一列特判: 若 `grid[i][0] == 1`, `dp[0] = 0` 一次设了之后再也不会变回非 0 (没人加它), 这是滚动数组天然成立的边界处理.

## Solution

=== "C++"
    === "v1: 二维 DP (Yang 原版)"
        ```cpp
        class Solution {
        public:
            int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
                int m = obstacleGrid.size(), n = obstacleGrid[0].size();
                vector<vector<int>> dp(m, vector<int>(n, 0));
                for (int i = 0; i < m; i++) {
                    if (obstacleGrid[i][0] == 1) break;            // 遇障碍, 后面全 0 (不用赋值, 初始就是 0)
                    dp[i][0] = 1;
                }
                for (int j = 0; j < n; j++) {
                    if (obstacleGrid[0][j] == 1) break;
                    dp[0][j] = 1;
                }
                for (int i = 1; i < m; i++) {
                    for (int j = 1; j < n; j++) {
                        if (obstacleGrid[i][j] == 1) {
                            dp[i][j] = 0;
                        } else {
                            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
                        }
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
            int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
                int m = obstacleGrid.size(), n = obstacleGrid[0].size();
                vector<int> dp(n, 0);
                dp[0] = (obstacleGrid[0][0] == 0);                 // 起点能站才置 1
                for (int i = 0; i < m; i++) {
                    for (int j = 0; j < n; j++) {
                        if (obstacleGrid[i][j] == 1) {
                            dp[j] = 0;                             // 障碍清零
                        } else if (j > 0) {
                            dp[j] += dp[j - 1];                    // 否则累加左边
                        }
                        // j==0 时无操作: dp[0] 由上一行继承, 一旦清零再也不变
                    }
                }
                return dp[n - 1];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def uniquePathsWithObstacles(self, obstacleGrid: list[list[int]]) -> int:
            m, n = len(obstacleGrid), len(obstacleGrid[0])
            # 一维滚动, 跟 0062 同套路, 多一句障碍清零
            dp = [0] * n
            dp[0] = 1 - obstacleGrid[0][0]                         # 起点 0/1 翻转: 障碍则 0, 否则 1
            for i in range(m):
                for j in range(n):
                    if obstacleGrid[i][j] == 1:
                        dp[j] = 0
                    elif j > 0:
                        dp[j] += dp[j - 1]
            return dp[-1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} obstacleGrid
     * @return {number}
     */
    var uniquePathsWithObstacles = function(obstacleGrid) {
        const m = obstacleGrid.length, n = obstacleGrid[0].length;
        const dp = new Array(n).fill(0);
        dp[0] = 1 - obstacleGrid[0][0];                            // 起点障碍则 0
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (obstacleGrid[i][j] === 1) {
                    dp[j] = 0;
                } else if (j > 0) {
                    dp[j] += dp[j - 1];
                }
            }
        }
        return dp[n - 1];
    };
    ```

## Complexity

- **Time**: O(m × n).
- **Space**: O(m × n) (v1) / O(n) (v2).

## 相关题目

- [0062. Unique Paths](../0062-unique-paths/README.md) — 无障碍母题
- 0064\. Minimum Path Sum (待补) — 同网格, 求最小代价
- 0980\. Unique Paths III (待补) — 必须经过所有空格, 转向回溯
- 1289\. Minimum Falling Path Sum II (待补) — 网格 DP 变种

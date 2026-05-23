# 0509. Fibonacci Number / 斐波那契数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, Recursion, Math · 动态规划, 递归, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/fibonacci-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Compute `F(n)` where `F(0) = 0`, `F(1) = 1`, `F(n) = F(n-1) + F(n-2)`.

**中文**: 求斐波那契数 `F(n)`, 其中 `F(0) = 0`, `F(1) = 1`, `F(n) = F(n-1) + F(n-2)`.

## Key Insights

1. **DP 五步法 (Carl 模板) / Five-step DP template**

    | 步骤 | 本题 |
    |---|---|
    | ① dp 数组定义 | `dp[i]` = 第 i 个斐波那契数 |
    | ② 递推公式 | `dp[i] = dp[i-1] + dp[i-2]` |
    | ③ 初始化 | `dp[0] = 0`, `dp[1] = 1` |
    | ④ 遍历顺序 | 从小到大 (i 依赖更小的 i) |
    | ⑤ 举例推导 | 0 1 1 2 3 5 8 13 ... |

    > 入门 DP 用来熟练流程, 后面所有 DP 都套这五步.

2. **滚动变量优化空间 / Rolling pair → O(1) space**

    `dp[i]` 只依赖 `dp[i-1]` 和 `dp[i-2]`, 不需要保留整个数组. 用两个变量 `dp[0]`, `dp[1]` 滚动:

    ```
    sum = dp[0] + dp[1]
    dp[0] = dp[1]          // 旧的 dp[1] 变成新的 dp[i-2]
    dp[1] = sum            // 新算出的变成 dp[i-1]
    ```

    > 所有"只依赖最近 k 个状态"的 DP 都能这样压. 后面 0070 爬楼梯, 0746 最小花费爬楼梯同模板.

3. **边界 `n <= 1` 必须先返回 / Guard before allocating**

    若 n = 0 或 1, `dp[1] = 1` 这一句会越界 (n=0 时 `vector<int>(1)` 没有索引 1). 直接 `return n` 短路.

## Solution

=== "C++"
    === "v2 推荐: 滚动变量 O(1) 空间"
        ```cpp
        class Solution {
        public:
            int fib(int n) {
                if (n <= 1) return n;
                int dp[2] = {0, 1};
                for (int i = 2; i <= n; i++) {
                    int sum = dp[0] + dp[1];
                    dp[0] = dp[1];                                 // 滚动: 旧 dp[1] 变 dp[i-2]
                    dp[1] = sum;                                   // 新值变 dp[i-1]
                }
                return dp[1];
            }
        };
        ```

    === "v1: 完整 dp 数组"
        ```cpp
        class Solution {
        public:
            int fib(int n) {
                if (n <= 1) return n;
                vector<int> dp(n + 1, 0);
                dp[0] = 0;
                dp[1] = 1;
                for (int i = 2; i <= n; i++) {
                    dp[i] = dp[i - 1] + dp[i - 2];
                }
                return dp[n];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def fib(self, n: int) -> int:
            if n <= 1:
                return n
            # 元组解包一行完成滚动: 右边先全部求值再赋值给左边
            # 等价 C++: int sum = a+b; a = b; b = sum; (但少一个临时变量)
            a, b = 0, 1
            for _ in range(2, n + 1):
                a, b = b, a + b
            return b
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var fib = function(n) {
        if (n <= 1) return n;
        // 解构赋值 [a, b] = [b, a+b]: 右边是临时数组, 先算完再解构, 模拟 Python 元组交换
        let [a, b] = [0, 1];
        for (let i = 2; i <= n; i++) {
            [a, b] = [b, a + b];
        }
        return b;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍循环.
- **Space**: O(1) (v2 滚动) / O(n) (v1 完整数组).

## 相关题目

- 0070\. Climbing Stairs (待补) — 同滚动 DP, `dp[i] = dp[i-1] + dp[i-2]` 一模一样, 只是初值不同
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 同模板, 转移多一个 min
- 1137\. N-th Tribonacci Number (待补) — 滚动三变量

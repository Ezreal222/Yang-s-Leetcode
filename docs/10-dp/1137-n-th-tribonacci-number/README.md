# 1137. N-th Tribonacci Number / 第 N 个泰波那契数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: DP, Linear · 动态规划, 线性
    - **Link**: [LeetCode](https://leetcode.com/problems/n-th-tribonacci-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **T(n) = T(n-1) + T(n-2) + T(n-3)**, T(0)=0, T(1)=T(2)=1 → linear DP; can optimize to **O(1) space with 3 rolling vars**.
>
> **中文**: **T(n) = T(n-1) + T(n-2) + T(n-3)**, T(0)=0, T(1)=T(2)=1 → 一维 DP; 空间可优化到 **O(1) 3 变量滚动**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 泰波那契数列: T(0)=0, T(1)=1, T(2)=1, T(n) = T(n-1) + T(n-2) + T(n-3). 求 T(n).

**中文**: Fibonacci 三项版, 求 T(n).

## Key Insights

1. **🔑 一维 DP 最简模板 / Simplest 1D DP**

    跟 [0509 Fibonacci](../0509-fibonacci-number/README.md) 一样, 就多一项累加. **base**: T(0)=0, T(1)=T(2)=1. **递推**: dp[i] = dp[i-1] + dp[i-2] + dp[i-3].

    > **"给递推式 + base"** = DP 送分题模板. 面试暖场必备.

2. **🔑 空间优化 O(1): 3 变量滚动 / 3-var rolling**

    只需保**最近 3 项**:

    ```cpp
    int a = 0, b = 1, c = 1;
    for (int i = 3; i <= n; i++) {
        int d = a + b + c;                   // 新项
        a = b; b = c; c = d;                 // 滚动
    }
    return c;
    ```

    - **`a, b, c`** 分别对应 T(i-3), T(i-2), T(i-1).
    - **`d = a + b + c`** = T(i).
    - **滚动**: 三个变量向左移一位.

    > **"依赖最近 k 项 → 滚动 k 变量 O(k) 空间"** — 跟 [0198 House Robber](../0198-house-robber/README.md) / [0509 Fibonacci](../0509-fibonacci-number/README.md) 同源.

3. **🔑 早退: `n <= 1` 直接返 n / Early return for base**

    - `n = 0` → 返 0.
    - `n = 1` → 返 1.
    - 免建 dp 数组浪费.

4. **🔑 复杂度 / Complexity**

    | | Time | Space |
    |---|---|---|
    | 数组版 | O(n) | O(n) |
    | **滚动版** | **O(n)** | **O(1)** |

    > **数据无界 (n ≤ 37 per LC) 时 O(1) 更优雅**. 数组版可读性略好.

## Solution

=== "C++"

    **v1: 数组版**

    ```cpp
    class Solution {
    public:
        int tribonacci(int n) {
            if (n <= 1) return n;
            vector<int> dp(n + 1, 0);
            dp[1] = 1; dp[2] = 1;
            for (int i = 3; i <= n; i++) {
                dp[i] = dp[i - 1] + dp[i - 2] + dp[i - 3];
            }
            return dp[n];
        }
    };
    ```

    **v2: 3 变量滚动 O(1) 空间**

    ```cpp
    class Solution {
    public:
        int tribonacci(int n) {
            if (n <= 1) return n;
            if (n == 2) return 1;
            int a = 0, b = 1, c = 1;
            for (int i = 3; i <= n; i++) {
                int d = a + b + c;
                a = b; b = c; c = d;
            }
            return c;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def tribonacci(self, n: int) -> int:
            if n <= 1: return n
            if n == 2: return 1
            # 元组解包滚动 — 右侧先算完再赋左, 一行搞定三滚
            # 跟数组版等价但更 Pythonic
            a, b, c = 0, 1, 1
            for _ in range(3, n + 1):
                a, b, c = b, c, a + b + c
            return c
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var tribonacci = function(n) {
        if (n <= 1) return n;
        if (n === 2) return 1;
        let a = 0, b = 1, c = 1;
        for (let i = 3; i <= n; i++) {
            // JS 无 tuple 解包到局部变量的一步语法, 用 tmp
            // (或数组解构: [a, b, c] = [b, c, a + b + c]; — 但要注意 [x] = [y] 是数组构造/解构, 略慢)
            const d = a + b + c;
            a = b; b = c; c = d;
        }
        return c;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) 数组版, **O(1)** 滚动版.

## 相关题目

- [0509. Fibonacci Number](../0509-fibonacci-number/README.md) — 二项递推母题
- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — Fibonacci 变形
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 花费型
- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP + 选/不选
- [0091. Decode Ways](../0091-decode-ways/README.md) — 线性 DP + 双转移
- [0983. Minimum Cost For Tickets](../0983-minimum-cost-for-tickets/README.md) — 线性 DP + 三种选择
- 0873\. Length of Longest Fibonacci Subsequence (待补) — Fibonacci 序列 DP
- 0842\. Split Array into Fibonacci Sequence (待补) — 回溯 + Fibonacci

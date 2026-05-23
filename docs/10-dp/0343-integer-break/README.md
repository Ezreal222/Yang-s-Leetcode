# 0343. Integer Break / 整数拆分

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Math, Greedy · 动态规划, 数学, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/integer-break/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 把正整数 `n` (n ≥ 2) 拆成**至少两个**正整数的和, 求这些数**乘积的最大值**.

**中文**: 把 `n` 拆成至少两个正整数之和, 求乘积最大值.

## Key Insights

1. **DP 状态: dp[i] = 拆 i 能得的最大乘积 / dp[i] = max product breaking i**

    转移分两种切法 (枚举切下来的第一块 `j`):

    - **只切两刀**: 直接 `j × (i-j)` — `i-j` 不再拆.
    - **切两刀以上**: `j × dp[i-j]` — `i-j` 继续递归拆.

    取最大:

    $$dp[i] = \max_{1 \le j < i} \big( \max(j \times (i-j),\ j \times dp[i-j]) \big)$$

    > 必须两个都考虑 — 因为 `dp[i-j]` 默认拆了, 可能比不拆 `i-j` 小 (例如 `dp[3] = 2` < 3 本身).

2. **为什么 `j` 不用枚举 i/2 以上 / Symmetry — j up to i/2 is enough**

    `j × (i-j)` 关于 `j` 对称, 枚到 `i/2` 就够. Yang 写到 `i-1` 是冗余但正确 — 复杂度仍 O(n²). 优化版可以提前 break.

3. **初始化 `dp[2] = 1` / Base case**

    `dp[0]`, `dp[1]` 没有"拆成至少两个" 的意义, 留 0 不参与转移. `dp[2] = 1` (只能 1+1). 从 `i=3` 开始填.

4. **数学贪心: 尽量拆 3 / Greedy — split into 3s**

    数学上, 把 n 拆成等量小份, 单份越接近 `e ≈ 2.718` 乘积越大. 整数里最接近 e 的是 **3**, 其次 2.

    **证明拆 3 优于拆 2**: 6 = 3+3 → 9; 6 = 2+2+2 → 8. 6 = 2+4 → 8. 都不如两个 3.
    **证明 ≥ 5 必须拆**: 对 `x ≥ 5`, `3 × (x-3) = 3x-9 > x` ⟺ `2x > 9` ⟺ `x ≥ 5`. ✓

    余数处理:

    | n % 3 | 怎么拆 | 乘积 |
    |---|---|---|
    | 0 | 全 3 | `3^(n/3)` |
    | 1 | 留一个 3 + 1 凑成 `3+1 → 改成 2+2 或 4` (4 > 3+1=4 一样, 但 `4 > 2×2`) | `3^(n/3-1) × 4` |
    | 2 | 全 3 + 一个 2 | `3^(n/3) × 2` |

    > **n%3==1** 关键: 不能简单 `3^k × 1` (乘 1 等于不乘). 退一步把 `3+1 = 4` 当一份, 因为 `4 > 3×1`.

5. **特例 `n = 2, 3` 不能用贪心 / Edge cases**

    `n=2` → 1+1, 答案 1 (不是 2, 因为必须拆).
    `n=3` → 1+2, 答案 2 (不是 3).

    贪心从 n=4 起才合理 — Yang 用 `while (n > 4)` 把 n 降到 [2..4] 再统一乘. 当 n 降到 4: `res *= 4`; 降到 2: `res *= 2`; 降到 3: `res *= 3`. 都正确.

## Solution

=== "C++"
    === "v1: DP O(n²) (通用)"
        ```cpp
        class Solution {
        public:
            int integerBreak(int n) {
                vector<int> dp(n + 1, 0);
                dp[2] = 1;                                         // 唯一基态: 1+1
                for (int i = 3; i <= n; i++) {
                    for (int j = 1; j < i; j++) {
                        // 切两刀 vs 切多刀 都要考虑
                        dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]));
                    }
                }
                return dp[n];
            }
        };
        ```

    === "v2: 数学贪心 O(log n)"
        ```cpp
        class Solution {
        public:
            int integerBreak(int n) {
                if (n == 2) return 1;
                if (n == 3) return 2;
                int res = 1;
                while (n > 4) {                                    // 每次抠一个 3, 剩 ≤4 时统一处理
                    res *= 3;
                    n -= 3;
                }
                res *= n;                                          // n ∈ {2,3,4}, 直接乘
                return res;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def integerBreak(self, n: int) -> int:
            # DP 版: 跟 C++ 一一对应
            dp = [0] * (n + 1)
            dp[2] = 1
            for i in range(3, n + 1):
                # max(...) 接散参; 用 generator 表达式一行算所有 j 的最大值
                # 等价 C++ 的双 max 嵌套
                dp[i] = max(max(j * (i - j), j * dp[i - j]) for j in range(1, i))
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var integerBreak = function(n) {
        // 数学贪心版 — 短且 O(log n)
        if (n === 2) return 1;
        if (n === 3) return 2;
        let res = 1;
        while (n > 4) {
            res *= 3;
            n -= 3;
        }
        return res * n;
    };
    ```

## Complexity

- **Time**: O(n²) (v1 DP) / O(n) (v2 贪心, 循环最多 n/3 次).
- **Space**: O(n) (v1) / O(1) (v2).

## 相关题目

- [0096. Unique Binary Search Trees](../0096-unique-binary-search-trees/README.md) — 同"枚举切点 + 子问题相乘" 结构 (卡特兰数)
- 0279\. Perfect Squares (待补) — 类似"拆 n 求最优", 但拆成平方数之和求最少个数
- 0264\. Ugly Number II (待补) — 数学贪心 + 多指针

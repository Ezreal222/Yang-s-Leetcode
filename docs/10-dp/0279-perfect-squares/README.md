# 0279. Perfect Squares / 完全平方数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack, Math · 动态规划, 完全背包, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/perfect-squares/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给整数 `n`, 求**和为 `n` 的完全平方数 (1, 4, 9, 16, ...) 的最少个数**.

**中文**: 给 `n`, 求和为 `n` 的最少完全平方数个数.

## Key Insights

1. **就是 [0322 Coin Change](../0322-coin-change/README.md) 的孪生题 / Same as Coin Change with squares as coins**

    把"硬币" 换成"完全平方数 `1, 4, 9, ..., ⌊√n⌋²`", 求最少凑出 `n` — 一字不差套完全背包 min 模板:

    | | 0322 | **本题** |
    |---|---|---|
    | 物品集 | 输入数组 `coins` | **动态生成** `1², 2², ..., ⌊√n⌋²` |
    | 目标 | amount | n |
    | 转移 | `min(dp[j], dp[j-coin]+1)` | `min(dp[j], dp[j-i²]+1)` |
    | 边界 | 凑不出返 -1 | **永远凑得出** (Lagrange 四平方定理) |

2. **物品动态生成: 外层 `i*i ≤ n` / Generate squares on the fly**

    不需要先把所有平方数填到 vector — 外层 `for (int i = 1; i*i <= n; i++)` 直接生成. 一共 `⌊√n⌋` 个物品.

    > 完全背包"物品集合不大且可计算" 时常用这套省内存写法.

3. **🔑 Lagrange 四平方定理: 答案 ≤ 4 / Answer is always 1, 2, 3, or 4**

    任何正整数都能写成不超过 4 个完全平方数之和 ([Lagrange's four-square theorem](https://en.wikipedia.org/wiki/Lagrange%27s_four-square_theorem)). 所以:

    - 不需要 `-1` 返回值的容错.
    - DP 答案上界是 4 → 可以用更紧的哨兵 `n+1` 或 `5` 替代 `INT_MAX`.

    **数学速判 O(1)** (进阶, 非必需):

    - `n = k²` → 1.
    - `n = a² + b²` 检查 → 2.
    - Legendre 三平方定理: `n` 不能写成 `4^a × (8b + 7)` → 3, 否则 4.

    > LC 测试规模下 DP 完全够用, 这里只是提一嘴有 O(1) 数学解.

4. **`dp[0] = 0` + INT_MAX 哨兵 + 防溢出 / Same boilerplate as 0322**

    跟 0322 完全一样:

    - `dp[0] = 0`: 凑出 0 需要 0 个平方数.
    - `INT_MAX` 标记不可达 (本题用不上, 因为 Lagrange 保证可达, 但模板保留).
    - 转移前 `if (dp[j-square] != INT_MAX)` 防 `+1` 溢出.

5. **正序 j: 完全背包标配 / Forward j, items reusable**

    每个平方数可以用任意次 (例如 `12 = 4 + 4 + 4`, 三个 `4`) → 正序 `j: square → n`.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numSquares(int n) {
            vector<int> dp(n + 1, INT_MAX);
            dp[0] = 0;
            for (int i = 1; i * i <= n; i++) {                     // 物品: 1², 2², ..., ⌊√n⌋²
                int square = i * i;
                for (int j = square; j <= n; j++) {                // 正序: 完全背包
                    if (dp[j - square] != INT_MAX) {
                        dp[j] = min(dp[j], dp[j - square] + 1);
                    }
                }
            }
            return dp[n];                                          // Lagrange 保证 ≤ 4, 一定可达
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def numSquares(self, n: int) -> int:
            # float('inf') 比 INT_MAX 更干净: Python int 无大小限制, inf+1 仍是 inf 不溢出
            dp = [float('inf')] * (n + 1)
            dp[0] = 0
            # range(1, int(n**0.5) + 1): 物品 1..√n. 用整数次方 i*i 避免浮点误差
            i = 1
            while i * i <= n:
                sq = i * i
                for j in range(sq, n + 1):                         # 正序: 完全背包
                    dp[j] = min(dp[j], dp[j - sq] + 1)
                i += 1
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var numSquares = function(n) {
        // n+1 当上界哨兵 — Lagrange 保证 ≤ 4, n+1 绰绰有余
        const dp = new Array(n + 1).fill(n + 1);
        dp[0] = 0;
        for (let i = 1; i * i <= n; i++) {
            const sq = i * i;
            for (let j = sq; j <= n; j++) {
                dp[j] = Math.min(dp[j], dp[j - sq] + 1);
            }
        }
        return dp[n];
    };
    ```

## Complexity

- **Time**: O(n × √n) — 外层 √n 个物品, 内层 n.
- **Space**: O(n).

## 相关题目

- [0322. Coin Change](../0322-coin-change/README.md) — 完全模板, 本题只是把 coins 换成平方数
- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 完全背包组合数版本
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 完全背包排列数版本
- 1449\. Form Largest Integer With Digits That Add up to Target (待补) — 完全背包 + 字典序最大字符串
- 0983\. Minimum Cost For Tickets (待补) — 完全背包 min 变体
- [0139. Word Break](../0139-word-break/README.md) — 完全背包判定 (能否拼接出字符串)

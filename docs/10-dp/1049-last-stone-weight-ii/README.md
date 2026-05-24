# 1049. Last Stone Weight II / 最后一块石头的重量 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, 0/1 Knapsack · 动态规划, 01 背包
    - **Link**: [LeetCode](https://leetcode.com/problems/last-stone-weight-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 一堆石头, 每次任挑两块 `a ≥ b` 撞, 撞完剩 `a - b` (若相等都没了). 重复直到 ≤ 1 块, 求最后剩重 (没剩则 0).

**中文**: 每次挑两块石头相撞 `a - b`, 一直撞到剩 ≤ 1 块, 求最小剩重.

## Key Insights

1. **核心转化: 撞 = 给每块石头分 + 或 − 号 / Each collision = assign ± sign**

    每次 `a - b` 等价于"a 留正号, b 加负号合并到 a". 几轮下来, 整个过程等价于:

    $$\text{最终剩重} = \Big| \sum \pm s_i \Big|$$

    任意一种"分 ± 号" 的方案都可以通过合适的撞击顺序实现 (实质上是把符号"加"那部分捏成一堆, "减" 那部分捏成另一堆, 最后两堆一撞).

    > 看到"两两操作消减" 的题, 先想能不能转成"全局符号分配". 这是这类题最重要的思维.

2. **进一步转化: 等价于"分两堆, 让差最小" / Split into two piles, minimize diff**

    设 "+ 号那组" 和 = `S₁`, "- 号那组" 和 = `S₂`, 则:

    - `S₁ + S₂ = total` (定值)
    - 答案 = `|S₁ - S₂|`

    令 `S₁ ≤ S₂` (即 `S₁ ≤ total/2`), 则:

    $$\text{答案} = (\text{total} - S_1) - S_1 = \text{total} - 2 S_1$$

    要最小化, 就最大化 `S₁`. 而 `S₁` 是"从 stones 里挑子集" 的和, 上限是 `total/2`. ⇒ **0/1 背包: 容量 `total/2`, 求最大可装价值**.

3. **跟 [0416](../0416-partition-equal-subset-sum/README.md) 是同一模板, 只是返回不同 / Same template as 0416, different return**

    | 题 | 问 | 返回 |
    |---|---|---|
    | 0416 | 能否分两堆**相等** | `dp[target] == target` |
    | 1049 | 分两堆差**最小** | `total - 2 * dp[half]` |

    DP 代码**一字不差**, 只换返回式. 0416 是判定题, 1049 是优化题, 但底层都是"凑最接近 total/2 的子集".

4. **不需要 `total % 2 != 0` 早返 / Don't need parity check**

    跟 0416 不同, 这题不要求和相等 — 奇数 total 也有意义 (`half = total / 2` 整除, 凑出的 `dp[half]` 会比 half 小, 最终剩 1+ ≥ 1). 直接进 DP 即可.

5. **`half = total / 2` 向下取整安全 / Floor is fine here**

    即便 total 是奇数, 向下取整丢掉的那 0.5 自动落入"被减堆" — 不影响最优解. 因为 `dp[half] ≤ half ≤ total/2`, `total - 2*dp[half] ≥ 0` 永远成立.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int lastStoneWeightII(vector<int>& stones) {
            int total = accumulate(stones.begin(), stones.end(), 0);
            int half = total / 2;
            vector<int> dp(half + 1, 0);
            for (int s : stones) {
                for (int j = half; j >= s; j--) {                  // 倒序: 0/1 背包
                    dp[j] = max(dp[j], dp[j - s] + s);
                }
            }
            return total - 2 * dp[half];                           // S₂ - S₁ = total - 2 S₁
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lastStoneWeightII(self, stones: list[int]) -> int:
            total = sum(stones)
            half = total // 2
            dp = [0] * (half + 1)
            for s in stones:
                # 倒序 — 跟 0416 同样的 0/1 背包规则
                for j in range(half, s - 1, -1):
                    dp[j] = max(dp[j], dp[j - s] + s)
            return total - 2 * dp[half]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} stones
     * @return {number}
     */
    var lastStoneWeightII = function(stones) {
        const total = stones.reduce((s, x) => s + x, 0);
        const half = total >> 1;                                   // 位运算等价 Math.floor(total/2), 对正数更快
        const dp = new Array(half + 1).fill(0);
        for (const s of stones) {
            for (let j = half; j >= s; j--) {
                dp[j] = Math.max(dp[j], dp[j - s] + s);
            }
        }
        return total - 2 * dp[half];
    };
    ```

## Complexity

- **Time**: O(n × sum/2).
- **Space**: O(sum/2).

## 相关题目

- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 同模板, 判定相等; 本题求最小差
- [0494. Target Sum](../0494-target-sum/README.md) — 同"分 ± 号" 模型, 求方案数 (背包计数版)
- 0474\. Ones and Zeroes (待补) — 二维容量 0/1 背包
- 1046\. Last Stone Weight (待补) — 简单版, 堆 + 模拟 (跟 1049 思路完全不一样, 注意区分)

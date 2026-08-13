# 0740. Delete and Earn / 删除并获得点数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Linear, House Robber Variant · 动态规划, 线性, 打家劫舍变体
    - **Link**: [LeetCode](https://leetcode.com/problems/delete-and-earn/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Pick x earns x × count(x) but deletes all x-1 and x+1** ⇒ **House Robber on value line**: build `points[v] = v × count(v)`, run rob DP on `points[0..maxVal]` (adjacent values can't both be picked).
>
> **中文**: **选值 x 得 x × 次数, 但顺带删 x±1** ⇒ **值域上跑打家劫舍**: 建 `points[v] = v × count(v)`, 在 `[0, maxVal]` 上跑 rob DP (相邻值不同选).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums`. 每步选一个值 `x` **获得 x 分**, 同时**删掉所有 x-1 和 x+1** (不能再选). 求最大总分.

- 例: `[3,4,2]` → 6 (选 4 得 4, 删 3 和 5(无); 再选 2 得 2. 或选 3 得 3, 删 2, 4).

**中文**: 选一个值得对应值 × 出现次数, 但会删掉 x±1 所有. 求最大总分.

## Key Insights

1. **🔑 灵魂化简: 转成 House Robber / Reduce to House Robber**

    **观察**:

    - 每个值 x 出现多次时, 一旦选 x 就**全选** (删所有 x-1 / x+1 反正不能选了, 剩下的 x 白拿). 所以**"选 x" = 得 `x × count(x)` 分**.
    - **"选 x 就不能选 x-1 和 x+1"** = 值域上**相邻不能同选** — 就是 [0198 House Robber](../0198-house-robber/README.md) 定义!

    → 化简成: **在值域 `[0, maxVal]` 上, `points[v] = v × count(v)`, 跑 rob DP**.

    > **"看似复杂 → 化简到经典模型"** 是解题的最高境界. 见到这题第一反应"rob 变体" 是能力标志.

2. **🔑 v1: 手动排序 + 分类 / v1: sort + branch by value gap**

    Yang 的**原始思路** — 不化简, 直接:

    - 按值排序**唯一值** 列表 (存 `(值, 次数)`).
    - 相邻两个值:
        - **差 > 1**: 独立, 都可选 → `dp[i] = dp[i-1] + cur`.
        - **差 = 1**: 冲突, 二选一 → `dp[i] = max(dp[i-1], dp[i-2] + cur)`.

    正确但**代码累赘**, 分两种情况判.

3. **🔑 v2: `sum[v]` 数组 + rob DP / v2: sum-array + classic rob DP**

    化简后**极简**:

    ```cpp
    vector<int> sum(maxVal + 1, 0);
    for (int x : nums) sum[x] += x;                          // sum[v] = v × count(v)
    // 然后在 sum[0..maxVal] 上跑 rob DP:
    int prev2 = 0, prev1 = 0;
    for (int x = 1; x <= maxVal; ++x) {
        int cur = max(prev1, prev2 + sum[x]);                // 经典 rob
        prev2 = prev1; prev1 = cur;
    }
    return prev1;
    ```

    - **`sum[x] += x`** — **每个出现的 x 加 x**. 出现 k 次 → sum[x] = k × x. 一行完成"值 → 总积分" 映射.
    - **rob DP**: `cur = max(不选此值 prev1, 选此值 prev2 + sum[x])`.
    - **值域连续遍历** (0..maxVal), 相邻值天然是 v 和 v+1 → 就是 House Robber 的语义.

    > **v1 → v2 是"化简后代码短一半"** 的教科书示范.

4. **🔑 v2 为啥不需要处理"某些值没出现"? / Why v2 handles missing values automatically**

    若值 v 从未出现, `sum[v] = 0` → 选或不选**都不影响 cur**. rob DP 自动跳过.

    > **"值域上 0 是无害的 no-op"** — 是这种化简能优雅的原因.

5. **🔑 复杂度 / Complexity**

    | | Time | Space |
    |---|---|---|
    | v1 排序 + 分类 | O(N log N) | O(K) K = 唯一值数 |
    | **v2 sum 数组 + rob** | **O(N + M)** M = maxVal | O(M) |

    - v2 若 **M 远大于 N** (值域稀疏但极值大), v1 反而好. 一般 v2 胜.

6. **🔑 跟 [0198 House Robber](../0198-house-robber/README.md) 关系 / Relation to 0198**

    | | 0198 House Robber | **0740 (本题)** |
    |---|---|---|
    | 相邻不选 | 数组下标相邻 | **值相邻 (差 1)** |
    | 每格价值 | `nums[i]` | **`v × count(v)`** |
    | DP | 一维 rob | **值域上一维 rob** |
    | 化简步骤 | 无 | **建 sum 数组** |

    > **一族两题**. 0740 = 0198 + 一层化简. 学一得二.

7. **🔑 空间进一步优化 O(1) / Further space to O(1)**

    v2 的 rob DP 部分只用 prev1, prev2 → O(1) 空间. **sum 数组 O(M) 无法省** (需要预计算).

    > 若 M 很大 (values 到 10^9) → v1 (排序 + 分类) 更省. v2 假设 M 合理 (≤ 10^4 per LC).

## Solution

=== "C++"

    **v2: sum + rob DP (推荐, 简洁化简)**

    ```cpp
    class Solution {
    public:
        int deleteAndEarn(vector<int>& nums) {
            int maxVal = *max_element(nums.begin(), nums.end());
            vector<int> sum(maxVal + 1, 0);
            for (int x : nums) sum[x] += x;                          // sum[v] = v × count(v)

            int prev2 = 0, prev1 = 0;
            for (int x = 1; x <= maxVal; ++x) {
                int cur = max(prev1, prev2 + sum[x]);                // 经典 rob
                prev2 = prev1;
                prev1 = cur;
            }
            return prev1;
        }
    };
    ```

    **v1: 排序 + 分类 (原始思路)**

    ```cpp
    class Solution {
    public:
        int deleteAndEarn(vector<int>& nums) {
            unordered_map<int, int> cnt;
            for (auto n : nums) cnt[n]++;
            vector<pair<int, int>> vec;
            for (auto [num, count] : cnt) vec.push_back({num, count});
            sort(vec.begin(), vec.end());
            int sz = vec.size();
            vector<int> dp(sz + 1, 0);
            dp[1] = vec[0].first * vec[0].second;
            for (int i = 2; i <= sz; i++) {
                int cur = vec[i - 1].first * vec[i - 1].second;
                if (vec[i - 1].first - vec[i - 2].first > 1) {
                    dp[i] = dp[i - 1] + cur;                          // 独立
                } else {
                    dp[i] = max(dp[i - 1], dp[i - 2] + cur);          // 冲突
                }
            }
            return dp[sz];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v2 化简版
        def deleteAndEarn(self, nums: list[int]) -> int:
            max_val = max(nums)
            # 一行建 sum 数组: sum[v] = v * count(v)
            # 也可 Counter(nums) + [c * v for v in range(max_val + 1)], 但 in-place 更快
            sums = [0] * (max_val + 1)
            for x in nums:
                sums[x] += x

            prev2 = prev1 = 0
            for s in sums[1:]:
                prev2, prev1 = prev1, max(prev1, prev2 + s)
            return prev1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var deleteAndEarn = function(nums) {
        // v2. Math.max(...nums) — spread. 大数组 stack overflow 时用 reduce
        const maxVal = Math.max(...nums);
        const sum = new Array(maxVal + 1).fill(0);
        for (const x of nums) sum[x] += x;

        let prev2 = 0, prev1 = 0;
        for (let x = 1; x <= maxVal; x++) {
            const cur = Math.max(prev1, prev2 + sum[x]);
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| v1 排序 + 分类 | O(N log N) | O(K) — K = 唯一值数 |
| **v2 sum + rob** | **O(N + M)** — M = maxVal | O(M) |

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — **化简目标, 母题**
- [0213. House Robber II](../0213-house-robber-ii/README.md) — 环形版
- [0337. House Robber III](../0337-house-robber-iii/README.md) — 树形版
- [3840. House Robber V](../3840-house-robber-v/README.md) — 变体
- [1137. N-th Tribonacci Number](../1137-n-th-tribonacci-number/README.md) — 线性 DP 三项
- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 一维 DP 基础
- 0790\. Domino and Tromino Tiling (待补) — 线性 DP + 状态机
- 1911\. Maximum Alternating Subsequence Sum (待补) — 交替子序列 DP

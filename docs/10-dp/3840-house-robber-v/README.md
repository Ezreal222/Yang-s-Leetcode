# 3840. House Robber V / 打家劫舍 V

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP · 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/house-robber-v/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 一排房子, `nums[i]` 是钱, `colors[i]` 是颜色. **相邻同色** 不能同时偷; 相邻不同色没限制. 求最多偷多少.

**中文**: 房子有钱 `nums[i]` 和颜色 `colors[i]`. 相邻**同色** 不能同时偷, 相邻**不同色** 可自由组合. 求最大总金额.

## Key Insights

1. **跟 [0198 House Robber](../0198-house-robber/README.md) 的差别: 约束改成"按颜色相邻" / Constraint is now color-based adjacency**

    0198 是**所有相邻不能同偷**; 本题只有**相邻同色** 才禁同偷, **相邻不同色** 反而可以都偷.

    | 题 | 约束 | 转移 (i 不为 0) |
    |---|---|---|
    | [0198](../0198-house-robber/README.md) | 所有相邻不可同偷 | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` |
    | **3840 (本题)** | 仅同色相邻不可同偷 | 分两种情况, 见 #3 |

    > 看到"加一维属性约束" 的 DP, 先想"约束是否影响转移结构, 还是只是分支判断". 本题是后者.

2. **状态: `dp[i] = 前 i+1 间房的最大金额` / Same as 0198, indexed by last considered house**

    `dp[i]` 不强制偷第 i 间 — 既包括"偷 i" 也包括"不偷 i" 两种配置的最优. 跟 0198 同语义.

3. **🔑 转移分两种 / Transition splits on color match**

    用"最后一步" 思维: 到达 `dp[i]` 的最后一步是"偷或不偷 i". 偷 i 时, 是否能让 i-1 也偷取决于颜色:

    - **`colors[i] == colors[i-1]`** (同色 → 同 0198 规则):
        - 偷 i: `dp[i-2] + nums[i]` (i-1 不能偷)
        - 不偷: `dp[i-1]`
        - 取 max: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`

    - **`colors[i] != colors[i-1]`** (不同色 → 无约束):
        - 偷 i: `dp[i-1] + nums[i]` (i-1 偷不偷都行, 用 dp[i-1] 的最优)
        - 不偷: `dp[i-1]`
        - 因 `nums[i] ≥ 0`, 偷必不劣 → `dp[i] = dp[i-1] + nums[i]`

    > 不同色那行省了 `max(dp[i-1], dp[i-1] + nums[i])` 因为 `nums[i] ≥ 0`. 这是题目保证正数才成立的小优化.

4. **`dp[1]` 也要按颜色分支 / Base case also color-aware**

    跟 0198 的 `max(nums[0], nums[1])` 不同:

    - 同色: `max(nums[0], nums[1])` (两间只能偷一间)
    - 不同色: `nums[0] + nums[1]` (两间可以都偷)

    > 这是 0198 → 3840 最容易漏的地方. 边界初始化必须跟转移规则保持一致.

5. **`long long` 防溢出 / Use long long**

    Yang 用了 `vector<long long> dp`. 如果 nums[i] 可达 10^4, n 可达 10^5, 总和上界 10^9 — 仍在 int 范围, 但 long long 更稳. 看具体题目数据约束.

6. **可滚动到 O(1) / Rolling pair optimization**

    `dp[i]` 只依赖 `dp[i-1]` 和 `dp[i-2]` → 两个变量足够. 跟 [0198](../0198-house-robber/README.md) / [0509](../0509-fibonacci-number/README.md) 同套路. 这里保留数组写法因为多分支判断, 数组更直观.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        long long rob(vector<int>& nums, vector<int>& colors) {
            int n = nums.size();
            if (n == 1) return nums[0];
            vector<long long> dp(n, 0);
            dp[0] = nums[0];
            // 边界也按颜色分支
            dp[1] = (colors[0] == colors[1])
                    ? max(nums[0], nums[1])
                    : (long long)nums[0] + nums[1];
            for (int i = 2; i < n; i++) {
                if (colors[i] == colors[i - 1]) {                  // 同色 → 0198 规则
                    dp[i] = max(dp[i - 1], dp[i - 2] + nums[i]);
                } else {                                           // 不同色 → 偷必不劣
                    dp[i] = dp[i - 1] + nums[i];
                }
            }
            return dp[n - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def rob(self, nums: list[int], colors: list[int]) -> int:
            n = len(nums)
            if n == 1:
                return nums[0]
            dp = [0] * n
            dp[0] = nums[0]
            # 三元 + 颜色判断
            dp[1] = max(nums[0], nums[1]) if colors[0] == colors[1] else nums[0] + nums[1]
            for i in range(2, n):
                if colors[i] == colors[i - 1]:
                    dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])    # 同色: 0198 规则
                else:
                    dp[i] = dp[i - 1] + nums[i]                    # 不同色: 偷必不劣
            return dp[n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number[]} colors
     * @return {number}
     */
    var rob = function(nums, colors) {
        const n = nums.length;
        if (n === 1) return nums[0];
        const dp = new Array(n).fill(0);
        dp[0] = nums[0];
        dp[1] = colors[0] === colors[1]
                ? Math.max(nums[0], nums[1])
                : nums[0] + nums[1];
        for (let i = 2; i < n; i++) {
            if (colors[i] === colors[i - 1]) {
                dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
            } else {
                dp[i] = dp[i - 1] + nums[i];
            }
        }
        return dp[n - 1];
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) (可压到 O(1)).

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 母题, "所有相邻不可同偷"
- [0213. House Robber II](../0213-house-robber-ii/README.md) — 环形版
- [0337. House Robber III](../0337-house-robber-iii/README.md) — 树形版
- [2560. House Robber IV](../../13-binary-search/2560-house-robber-iv/README.md) — 题名同系列, 但用二分答案 (在 §13)
- [§10 DP 思维流程](../topic-dp-thinking-process.md) — 加约束的 DP 通常不改状态, 只改转移分支

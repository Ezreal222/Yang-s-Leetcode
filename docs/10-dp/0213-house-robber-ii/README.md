# 0213. House Robber II / 打家劫舍 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP · 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/house-robber-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 跟 [0198](../0198-house-robber/README.md) 同, 但房子排成**环**: 第一间和最后一间也是相邻的. 求最多偷多少.

**中文**: 房子排成环, 首尾也相邻, 求最多偷多少.

## Key Insights

1. **🔑 拆环为链: 首尾互斥 → 两种线性子问题取 max / Break circle by splitting into two linear cases**

    首尾相邻意味着**最多偷其中一个**. 三种合法情况:

    - 偷首不偷尾
    - 不偷首偷尾
    - 首尾都不偷

    把 "**不偷尾**" 和 "**不偷首**" 两种线性切片各跑一次 [0198](../0198-house-robber/README.md), 取 max:

    | 切片 | 范围 | 涵盖 |
    |---|---|---|
    | A: 不偷尾 | `nums[0..n-2]` | "偷首不偷尾" + "首尾都不偷" |
    | B: 不偷首 | `nums[1..n-1]` | "不偷首偷尾" + "首尾都不偷" |

    `max(A, B)` 涵盖全部三种合法情况. "首尾都不偷" 在两边都能产生, 不会漏.

    > **拆解约束** 是 DP 经典技巧 — 环 → 两条链, 三状态 → 三次跑, 等等. 把复杂约束拆成已解决的子问题.

2. **复用 0198 — 抽成 `robLinear(start, end)` / Factor out linear robber**

    把 0198 的循环参数化成 `[start, end)`. 注意是**左闭右开**, 跟 STL 习惯一致, `end - start = 区间长度`. 写法干净.

3. **初始化 `prev2 = prev1 = 0` 比 [0198](../0198-house-robber/README.md) 原版更通用 / Zero-init is cleaner**

    Yang 在这里用 `prev2 = prev1 = 0`, 不是 `nums[start]`, `max(nums[start], nums[start+1])`. 含义: "**起点之前** 还没看任何房子 → 偷了 0 块". 第一次循环自然算出 `cur = max(0 + nums[start], 0) = nums[start]`.

    > 这种"虚拟前置 0" 写法**省了 n=1 的边界 if**, 也更适合 `robLinear` 通用化. 0198 也可以这么写, 但当时是为了凸显"`dp[1] = max(...)`" 的陷阱才显式写出.

4. **`n == 1` 必须特判 / Edge case n=1**

    n=1 时两个切片范围都是空 (`[0, 0)` 和 `[1, 1)`), `robLinear` 返回 0 → 漏掉那唯一一间房子. 必须 `if (n == 1) return nums[0]`.

5. **不需要"首尾都不偷" 单独考虑 / Don't double-handle the third case**

    "首尾都不偷" 在 A 和 B 切片里都是合法配置 (循环里"不偷" 是默认选项). 不用第三次跑.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int rob(vector<int>& nums) {
            int n = nums.size();
            if (n == 1) return nums[0];                            // 拆环切片对 n=1 失效
            return max(robLinear(nums, 0, n - 1),                  // 不偷尾
                       robLinear(nums, 1, n));                     // 不偷首
        }
    private:
        // 跑 nums[start..end-1] 上的 0198, 左闭右开
        int robLinear(vector<int>& nums, int start, int end) {
            int prev2 = 0, prev1 = 0;                              // 起点前虚拟 0
            for (int i = start; i < end; i++) {
                int cur = max(prev2 + nums[i], prev1);             // 偷 or 不偷
                prev2 = prev1;
                prev1 = cur;
            }
            return prev1;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def rob(self, nums: list[int]) -> int:
            n = len(nums)
            if n == 1:
                return nums[0]

            def rob_linear(arr):
                # 元组解包滚动, 跟 0198 同套路, 但虚拟 0 初值更通用
                prev2, prev1 = 0, 0
                for x in arr:
                    prev2, prev1 = prev1, max(prev1, prev2 + x)
                return prev1

            # 用切片就不需要传 start/end 参数, Pythonic
            return max(rob_linear(nums[:-1]), rob_linear(nums[1:]))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var rob = function(nums) {
        const n = nums.length;
        if (n === 1) return nums[0];

        const robLinear = (start, end) => {
            let prev2 = 0, prev1 = 0;
            for (let i = start; i < end; i++) {
                [prev2, prev1] = [prev1, Math.max(prev1, prev2 + nums[i])];
            }
            return prev1;
        };

        return Math.max(robLinear(0, n - 1), robLinear(1, n));
    };
    ```

## Complexity

- **Time**: O(n) — 跑两遍 0198.
- **Space**: O(1) — 滚动变量.

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性母题, 本题复用
- [0337. House Robber III](../0337-house-robber-iii/README.md) — 树形版, 后序 DFS + 双状态
- 0740\. Delete and Earn (待补) — 转化版: 按值分桶 + 0198
- 0918\. Maximum Sum Circular Subarray (待补) — 同款"环形 → 拆线性" 思路, 应用在最大子数组和
- [§10 DP 思维流程](../topic-dp-thinking-process.md) — 拆解约束 + 复用子问题的典型例子

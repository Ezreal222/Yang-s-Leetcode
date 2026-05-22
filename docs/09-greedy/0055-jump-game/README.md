# 0055. Jump Game / 跳跃游戏

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Array · 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/jump-game/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given an integer array `nums`, each element = maximum jump length from that position. Starting at index 0, return whether you can reach the last index.

**中文**: 给一个整数数组 `nums`, 每个元素表示从该位置最多能跳几步. 起点 index 0, 判断能否到末尾.

## 思路

### Core idea

**贪心维护"目前能到达的最远 index" `cover`**:

- 从 0 出发, for `i` 在 `[0, cover]` 里走 (i 不能超出可达范围).
- 每个 `i`, 用 `i + nums[i]` 更新 cover.
- 一旦 `cover >= n - 1` → 能到末尾, return true.
- 循环结束 cover 还不够 → return false.

### Key Insights

1. **`i <= cover` 是这题的灵魂 / Loop bound = current reach**

    for 条件**不是** `i < n`, 而是 `i <= cover`. 含义: i 只在"已经能到达的范围" 里扩展. 写 `i < n` 等于"假设所有位置都能到", 错: 比如 `[0, 2, 3, 1, 4]`, i=0 时 cover=0, 永远不该用 i=1 的 nums[1]=2 来扩展 (因为 0 跳不到 1).

    这是 Carl 反复强调的"小心思", 写错会 AC 错误答案.

2. **不用真的找具体跳法 / Greedy doesn't need explicit jumps**

    传统思路: 维护"当前位置" 一步一步跳, 选哪步? 贪心放下这个心智负担 — **只管"能到达的范围"**, 不管具体路径. 等价证明: 如果 cover 包含 n-1, 总能找到一条从 0 跳到 n-1 的路径 (反推).

3. **`cover >= n - 1` 内层早停 / Early return**

    一旦 cover 覆盖末尾, 直接 true. 不早停也对, 但浪费时间. Yang 加了, 学回去.

4. **DP 替代 / DP alternative is O(n²), don't use**

    `dp[i]` = bool 能否到 i. 转移 `dp[i] = ∃j<i: dp[j] && j + nums[j] >= i`. O(n²) — 比贪心 O(n) 差一档. 仅作为对比.

5. **跟 [0045 Jump Game II 求最少步数] / Sister problem**

    0045 求最少跳几次, 加一层"当前层最远" 跟踪 (BFS 层思路). 同款贪心骨架, 多一个状态. 0055 → 0045 是经典进阶.

### 一句话总结

**贪心: cover = 当前可达最远 index. for i 在 [0, cover] 扩展 `cover = max(cover, i + nums[i])`. cover ≥ n-1 就 true.**

### 图解

`nums = [2, 3, 1, 1, 4]`:

```
i=0  nums[0]=2  cover = max(0, 0+2) = 2
i=1  nums[1]=3  cover = max(2, 1+3) = 4  ≥ n-1=4 → true
```

`nums = [3, 2, 1, 0, 4]`:

```
i=0  nums[0]=3  cover = max(0, 0+3) = 3
i=1  nums[1]=2  cover = max(3, 1+2) = 3
i=2  nums[2]=1  cover = max(3, 2+1) = 3
i=3  nums[3]=0  cover = max(3, 3+0) = 3
i=4 出界 (i > cover=3) → 循环结束 → false
```

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool canJump(vector<int>& nums) {
            int cover = 0;
            for (int i = 0; i <= cover; i++) {                  // i 不能超过当前可达范围 — 关键
                cover = max(cover, i + nums[i]);
                if (cover >= (int)nums.size() - 1) return true; // 早停
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def canJump(self, nums: list[int]) -> bool:
            cover = 0
            i = 0
            # while + 显式 i 推进, 比 for-range 灵活 (因为 cover 在循环里会变)
            while i <= cover:
                cover = max(cover, i + nums[i])
                if cover >= len(nums) - 1:
                    return True
                i += 1
            return False
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {boolean}
     */
    var canJump = function(nums) {
        let cover = 0;
        // for 条件每轮重新求值, JS / C++ 都允许 cover 在循环里动态变化
        for (let i = 0; i <= cover; i++) {
            cover = Math.max(cover, i + nums[i]);
            if (cover >= nums.length - 1) return true;
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(n) — 最坏一遍扫.
- **Space**: O(1).

## 易错点

- **for 条件必须 `i <= cover` 不是 `i < n`**: 这是这题最容易写错的一行. 用 `i < n` 会从不可达位置扩展 cover, 答案错. 这是 Carl 这题反复强调的关键.
- **不要写 Python `for i in range(len(nums))`**: Python for-range 把范围在循环开始时固定, cover 后续变化没用. 必须 while + 显式 i++.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门
- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) / [0376](../0376-wiggle-subsequence/README.md) — 同款一遍贪心
- [0045. Jump Game II](../0045-jump-game-ii/README.md) — 求**最少**跳几步, 加一层"当前层最远" 跟踪
- 1306\. Jump Game III (待补) — 双向跳, BFS / DFS
- 0871\. Minimum Number of Refueling Stops (待补) — 贪心 + heap, 同思路 ("尽量晚加油")

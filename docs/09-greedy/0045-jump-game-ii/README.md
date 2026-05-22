# 0045. Jump Game II / 跳跃游戏 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, BFS, Array · 贪心, 广度优先搜索, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/jump-game-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a 0-indexed integer array `nums` (each element = max jump length from that position, guaranteed reachable), return the **minimum** number of jumps to reach the last index.

**中文**: 给一个非负整数数组 `nums` (每个元素 = 从该位置能跳的最远距离, 题目保证可达). 求到末尾的**最少跳跃次数**.

## 思路

### Core idea

**两个指针 + BFS 分层贪心** (Yang 自己的总结很到位):

- `curEnd` = **当前这一跳**能到达的最远位置 (这一层的右边界).
- `farthest` = **下一跳**能到达的最远位置 (这一层里所有 i 试探出来的).
- 扫每个 i, 不停更新 farthest. 当 `i == curEnd` (当前层走完), `jumps++` 并 `curEnd = farthest` (进下一层).

把每次跳跃当 BFS 一层就清晰: 同一层里的所有点 jumps 相同, 跨过 `curEnd` 才花一跳.

### Key Insights

1. **BFS 分层视角 / Think jumps as BFS layers**

    层 0 = 起点 `[0, 0]`. 层 1 = 从层 0 跳一步能到达的范围 `[1, nums[0]]`. 层 2 = 从层 1 任意点跳一步能到达的最远范围 (= farthest at end of 层 1). 这跟二叉树 BFS 的"按层处理" 一字不差.

    `curEnd` 就是当前层的最远 index; `farthest` 在扫这一层时累积, 一层结束时晋升为下一层的 `curEnd`.

2. **`i < nums.size() - 1` 防止多跳一次 / Stop just before the last index**

    末位 index 已经是终点, 走到那里就不用再跳了. 写成 `i < nums.size()` 会让循环包含最后一格, 触发额外一次"层切换" → jumps 多 1.

    例: `[2, 1]`, 答案是 1. 若扫到 i=1 时 i == curEnd=1 触发 jumps++, 结果会变成 2. 用 `n - 1` 排除末位.

3. **`i == curEnd` 才转层, 不要每步都 jumps++ / Only increment on layer crossing**

    每个 i 都跳一次是 0055 的 cover 思路, 这里不行: 一层里多个 i 不消耗多跳. **只在边界 i == curEnd 时算"花一跳进下一层"**.

4. **跟 [0055 Jump Game] 的对照 / Compared to single-pointer cover**

    | 题 | 状态 | 更新 |
    |---|---|---|
    | [0055](../0055-jump-game/README.md) (能否到) | 单 cover | for 每步 max, 终止判 cover ≥ n-1 |
    | **0045 (本题: 最少几跳)** | **curEnd + farthest** | for 每步 max farthest, 跨 curEnd → jumps++ |

    多一个变量 + 多一个 if = 从"能否" 变成"最少跳数". 这是贪心题中"加一层 BFS 跟踪" 的典型扩展.

5. **复杂度 O(n) / Same as Jump Game**

    一遍线性扫, 每个 index 只更新一次 farthest, 跨边界时 jumps++. 比 DP 解 (`dp[i] = min(dp[j]) + 1` for reaching i, O(n²)) 优一档.

### 一句话总结

**两指针: curEnd = 这一跳能到的最远, farthest = 下一跳能到的最远. for 扫每个 i 更新 farthest, 跨过 curEnd 就 jumps++. for 上限 n-1 防止多跳.**

### 图解

`nums = [2, 3, 1, 1, 4]`:

```
i=0: farthest = max(0, 0+2) = 2.  i == curEnd=0 → jumps=1, curEnd=2.
i=1: farthest = max(2, 1+3) = 4.
i=2: farthest = max(4, 2+1) = 4.  i == curEnd=2 → jumps=2, curEnd=4.
i=3: farthest = max(4, 3+1) = 4.  (i 还不到 n-1=4, 但循环到 i<4 就结束)
返回 2.   ← 0→1→4 共两跳
```

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int jump(vector<int>& nums) {
            int jumps = 0, curEnd = 0, farthest = 0;
            // i < n - 1: 末位无需再跳一次
            for (int i = 0; i < (int)nums.size() - 1; i++) {
                farthest = max(farthest, i + nums[i]);          // 下一跳的备选最远
                if (i == curEnd) {                              // 走完当前层
                    jumps++;
                    curEnd = farthest;                          // 晋升下一层
                }
            }
            return jumps;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def jump(self, nums: list[int]) -> int:
            jumps = cur_end = farthest = 0
            # range(len(nums) - 1): 末位不参与, 否则多跳一次
            for i in range(len(nums) - 1):
                farthest = max(farthest, i + nums[i])
                if i == cur_end:
                    jumps += 1
                    cur_end = farthest
            return jumps
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var jump = function(nums) {
        let jumps = 0, curEnd = 0, farthest = 0;
        for (let i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);
            if (i === curEnd) {
                jumps++;
                curEnd = farthest;
            }
        }
        return jumps;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 易错点

- **`i < n - 1` 不能 `i < n`**: 末位是终点, 不消耗一跳. 写 `i < n` 在 `[2, 1]` 这种 case 会算出 2 而不是 1.
- **`i == curEnd` 才 jumps++**: 同一层 multi-i 不算多跳. 写每步都 ++ 会大量超数.

## 相关题目

- [0055. Jump Game](../0055-jump-game/README.md) — 单 cover 求"能否到", 本题的"前置版"
- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) — 同款一遍贪心扫
- 1306\. Jump Game III (待补) — 双向跳, BFS / DFS
- 1345\. Jump Game IV (待补) — 加同值瞬移, BFS 经典
- 1696\. Jump Game VI (待补) — 求最大 score, 单调队列贪心
- 0871\. Minimum Number of Refueling Stops (待补) — 同款"分层最远 + 计数" 思路, 加 heap

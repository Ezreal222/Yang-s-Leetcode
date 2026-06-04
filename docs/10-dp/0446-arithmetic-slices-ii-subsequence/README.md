# 0446. Arithmetic Slices II - Subsequence / 等差数列划分 II - 子序列

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Hash Table, Sequence · 动态规划, 哈希表, 序列
    - **Link**: [LeetCode](https://leetcode.com/problems/arithmetic-slices-ii-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给数组 `nums`. 求**长度 ≥ 3 的等差子序列** 的数量 (按位置区分, 元素可不连续).

**中文**: 求长度 ≥ 3 的等差**子序列** 个数 (按下标区分).

## Key Insights

1. **🔑 状态: `dp[i][diff] = 以 i 结尾, 公差为 diff 的"弱等差子序列" (长度 ≥ 2) 个数` / Weak count with diff key**

    "弱" = 长度 ≥ 2 (含长度刚刚为 2 的"两点对"). 这是核心技巧 — **不直接数长度 ≥ 3 的**, 而是数"弱" 子序列, 在转移中**用"扩展" 的次数累加答案**.

    > 直接 DP 长度 ≥ 3 转移会丢信息 (需要知道前面有多少长度 = 2 的链可以扩). **退一步存"弱" 计数, 转移时自然累加**.

2. **🔑 `dp[i]` 不是定长数组, 是 `unordered_map<diff, count>` / Hash map per index**

    `diff` 可正可负可零, 值域很大 → 不能开二维数组. 每个 i 单独维护一个**哈希表** 存所有不同 diff 的计数.

    > **状态键的值域不固定时, 用哈希表代替数组维度** 是标准技巧.

3. **🔑 转移: 枚举 (j, i) 对, 累加扩展贡献 / Enumerate pairs (j, i)**

    对每对 `j < i`:

    - `diff = nums[i] - nums[j]` (用 long long 防溢出)
    - `cnt = dp[j].get(diff, 0)` — 已有多少"以 j 结尾, 公差 diff 的弱子序列"
    - **答案累加 `cnt`** — 每一条都能扩展到 i 形成**长度 ≥ 3** 的真等差子序列 (新加 i, 原长 ≥ 2 → 现长 ≥ 3)
    - **`dp[i][diff] += cnt + 1`**:
        - `cnt`: 扩展过来的
        - `+1`: 新建的 (j, i) 两点对 (长度 2 的弱子序列)

4. **🔑 ans 累加的是"扩展" 部分, 不算"两点对" / Why ans ignores +1**

    `+1` 是新建的长度 2 子序列 — **不是长度 ≥ 3, 不能加到 ans**. 只有 `cnt` 部分 (从 j 那边扩过来的) 才保证长度 ≥ 3.

    > 这是这道题最容易翻车的点. 写成 `ans += cnt + 1` 直接 WA.

5. **`long long` 防溢出 / Avoid int overflow**

    `nums[i] - nums[j]` 在两个值符号相反或绝对值很大时可能溢出 int. Yang 用 `long long diff` 防止. **数组元素可负**时尤其要小心.

6. **复杂度 O(n²) 时间, O(n²) 空间 / Quadratic**

    双循环 O(n²), 每个 i 的 map 最多 n 个 diff → 总 O(n²) 空间. LC 数据 n ≤ 1000, 10⁶ 操作完全可以.

7. **跟 [0300 LIS](../0300-longest-increasing-subsequence/README.md) 的关系 / vs LIS family**

    同款"**以 i 结尾 + 枚举 j < i**" 结构, 区别:

    | | [0300 LIS](../0300-longest-increasing-subsequence/README.md) | **0446 (本题)** |
    |---|---|---|
    | dp | `dp[i] = 长度` | `dp[i][diff] = 计数` |
    | 状态空间 | O(n) | O(n × \|diff\|) → 哈希表 |
    | 转移 | max | sum |
    | 答案 | max(dp[*]) | 累加 "扩展" 部分 |

    > **序列 DP 进阶**: 当"每个 i 还要按某属性细分" 时, 状态从一维数组升级到"数组 × 哈希表".

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numberOfArithmeticSlices(vector<int>& nums) {
            int n = nums.size();
            // 每个 i 一个 map: diff → 以 i 结尾的弱等差子序列计数
            vector<unordered_map<long long, int>> dp(n);
            int ans = 0;

            for (int i = 0; i < n; i++) {
                for (int j = 0; j < i; j++) {
                    long long diff = (long long)nums[i] - nums[j];     // 防溢出
                    int cnt = 0;
                    auto it = dp[j].find(diff);
                    if (it != dp[j].end()) cnt = it->second;           // 以 j 结尾的弱计数

                    ans += cnt;                                         // 扩展过来 → 长度 ≥ 3
                    dp[i][diff] += cnt + 1;                             // +1 是新两点对 (j, i)
                }
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def numberOfArithmeticSlices(self, nums: list[int]) -> int:
            n = len(nums)
            # defaultdict(int) 让 dp[i][diff] 默认 0, 不用 .get()
            dp = [defaultdict(int) for _ in range(n)]
            ans = 0

            for i in range(n):
                for j in range(i):
                    diff = nums[i] - nums[j]                            # Python int 无溢出
                    cnt = dp[j][diff]                                   # 弱计数, 不存默认 0
                    ans += cnt                                          # 扩展的就是长度 ≥ 3
                    dp[i][diff] += cnt + 1                              # +1 = 新两点对

            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var numberOfArithmeticSlices = function(nums) {
        const n = nums.length;
        const dp = Array.from({length: n}, () => new Map());
        let ans = 0;

        for (let i = 0; i < n; i++) {
            for (let j = 0; j < i; j++) {
                const diff = nums[i] - nums[j];                         // JS 数字范围够 (2^53)
                const cnt = dp[j].get(diff) ?? 0;
                ans += cnt;
                dp[i].set(diff, (dp[i].get(diff) ?? 0) + cnt + 1);
            }
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(n²) — 双循环, 哈希查 O(1) 摊销.
- **Space**: O(n²) — 每 i 的 map 最坏 n 条.

## 相关题目

- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 序列 DP, "以 i 结尾"
- 0413\. Arithmetic Slices (待补) — 等差**子数组** (连续), 简化版
- 0873\. Length of Longest Fibonacci Subsequence (待补) — 同款"以 (j, i) 对结尾" + 哈希
- 1027\. Longest Arithmetic Subsequence (待补) — 求最长等差子序列长度
- 1218\. Longest Arithmetic Subsequence of Given Difference (待补) — 给定 diff 求最长
- 1010\. Pairs of Songs With Total Durations Divisible by 60 (待补) — 哈希 + 配对

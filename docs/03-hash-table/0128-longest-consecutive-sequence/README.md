# 0128. Longest Consecutive Sequence / 最长连续序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Set, Amortized, Union-Find (alt) · 哈希集合, 摊销复杂度, 并查集 (备选)
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-consecutive-sequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Longest run of consecutive ints, O(n)** → dump into hash set, **only expand from sequence heads** (x with `x - 1` not in set), grow forward with `set.count(cur + 1)`. Each element visited ≤ 2 times → amortized O(n).
>
> **中文**: **求最长连续序列, O(n)** → 扔进 hash set, **只从"序列头"** (x - 1 不在集合的 x) 向后扩. 每元素被访问 ≤ 2 次 → 摊销 O(n).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给未排序数组 `nums`. 求**最长连续元素序列** 的长度. **要求 O(n)**.

**中文**: 无序数组求最长连续 (元素) 序列长度. 严格 O(n).

## Key Insights

1. **🔑 朴素思路 O(n log n): 排序 + 一遍扫 / Naive: sort then linear scan**

    排序后连续序列变成"相邻元素只差 1", 一遍扫累计. 简单, 但**违反 O(n) 要求**. Follow-up 才 O(n).

    > 面试若明确要 O(n), 排序方案要**主动否掉** 而不是先写 (显示知道限制).

2. **🔑 O(n) 关键 trick: 只从"序列头" 开始向后扩 / Only start from sequence heads**

    朴素想法: 对每个 `x`, 往后扩 `x, x+1, x+2, ...`, 记最长. **问题**: 若 x 在序列中间, 会**重复计算**同一序列的一部分 → O(n²).

    **修正**: 只有 `x` 是**序列头** (`x - 1` 不在集合) 时才向后扩. 判"是不是头" 只花 O(1):

    ```cpp
    if (s.count(x - 1)) continue;    // 不是头, 跳过
    // x 是头, 开始扩
    ```

    > **这一行 filter 是本题的灵魂**. 没它就 O(n²), 有它才 O(n).

3. **🔑 为啥总时间还是 O(n)? 摊销分析 / Why amortized O(n)**

    看起来嵌套了两层循环 (for + while), 像 O(n²). 但**每个元素**在 inner while 中**至多被访问一次** — 因为:

    - 每个整数属于**唯一一个连续序列**.
    - 只有该序列的**头** 起扩, 且**扩完整个序列**.

    → 所有 inner while 迭代加起来 ≤ n. 加上 outer for 的 O(n) → **总 O(n)**.

    > **摊销 (amortized)** 复杂度分析: **看总工作量**, 不看单次最坏. 常见在均摊 O(1) 的 vector push_back, Union-Find, 滑动窗口等.

4. **🔑 `unordered_set` vs `set` / Hash set vs tree set**

    | | `unordered_set` (Yang) | `set` |
    |---|---|---|
    | 查/插 | **O(1) 平均** | O(log n) |
    | 有序性 | 无 | 有 |
    | 本题需要 | ✅ | ❌ (排序反而拖累) |

    > **只用 lookup, 不用顺序** → 就选 hash set. 若用 `set`, 总时间变 O(n log n), 违反要求.

5. **🔑 从 unordered_set 迭代 = 从 unordered_map 迭代 / Iterating hash set**

    `for (int x : s)` 遍历 set. **顺序无关紧要** — 反正我们判"是否是头" 才决定动手. Yang 直接用 `for (int x : s)` 而不是 `for (int x : nums)` — 这样**天然去重**, 且省一层 O(1) 判 (nums 里可能有重复).

    > **小优化**: 用 set 而非 nums 迭代 = 天然 dedupe. 若用 nums, 也对, 但对重复元素会多做一次"是不是头" 检查.

6. **备选思路: Union-Find / Alternative: DSU**

    每个数与 `x + 1`, `x - 1` 合并入同一并查集组. 最后找最大组大小. **也 O(n α(n))** 摊销, 但代码 3× 长. 面试**知道**就行, 首选 hash set.

7. **复杂度 O(n) 时间, O(n) 空间 / Linear, linear**

    hash set 存 n 元素 → O(n) 空间. 摊销扫一遍 → O(n) 时间.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int longestConsecutive(vector<int>& nums) {
            unordered_set<int> s(nums.begin(), nums.end());          // 一次性装桶 + 去重
            int longest = 0;
            for (int x : s) {
                if (s.count(x - 1)) continue;                        // 不是"头" 就跳过, 灵魂 filter
                int cur = x, len = 1;
                while (s.count(cur + 1)) { cur++; len++; }           // 从头往后扩
                longest = max(longest, len);
            }
            return longest;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def longestConsecutive(self, nums: list[int]) -> int:
            # set(nums) 一步搞定去重 + O(1) lookup. 比 C++ 的 unordered_set 构造更省
            s = set(nums)
            longest = 0
            for x in s:
                # `x - 1 not in s` — Python 的 in 对 set 是 O(1) 平均, 跟 C++ .count 等价
                if x - 1 in s:
                    continue        # 不是头
                cur, length = x, 1
                # 手写 while — 也可用 while (cur := cur + 1) in s 加海象运算符, 但可读性变差
                while cur + 1 in s:
                    cur += 1
                    length += 1
                longest = max(longest, length)
            return longest
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var longestConsecutive = function(nums) {
        // new Set(iterable) 一步去重 + O(1) 平均查. 跟 C++ unordered_set / Python set 同源
        const s = new Set(nums);
        let longest = 0;
        // for..of 迭代 Set — 顺序不定但无所谓
        for (const x of s) {
            if (s.has(x - 1)) continue;   // 不是头, 跳
            let cur = x, len = 1;
            while (s.has(cur + 1)) { cur++; len++; }
            // Math.max 而不是三元 — 简洁
            longest = Math.max(longest, len);
        }
        return longest;
    };
    ```

## Complexity

- **Time**: **O(n)** 摊销 — 每元素在 inner while 中至多访问一次.
- **Space**: O(n) — hash set.

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 哈希集合基础
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 哈希分桶
- [0036. Valid Sudoku](../0036-valid-sudoku/README.md) — 多张 bool "哈希" 表
- 0300\. Longest Increasing Subsequence (待补) — 类似"最长" 但**不要求连续**, DP / 二分
- 0298\. Binary Tree Longest Consecutive Sequence (待补) — 树版
- 1218\. Longest Arithmetic Subsequence of Given Difference (待补) — 哈希 DP, 找定差数列
- 0674\. Longest Continuous Increasing Subsequence (待补) — 数组连续版, 一遍扫

# 0525. Contiguous Array / 连续数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Prefix Sum, Hash Map, Mapping Trick · 前缀和, 哈希表, 映射技巧
    - **Link**: [LeetCode](https://leetcode.com/problems/contiguous-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Longest contiguous subarray with equal 0s and 1s** → **mapping trick**: `0 → −1, 1 → +1`, so the problem becomes **longest subarray with sum 0** ⇔ two prefix sums equal. Hash first-index-per-sum, take `max(i − stored)`. Seed `idx[0] = −1` so a full-prefix match gives length `i + 1`.
>
> **中文**: **求 0 和 1 数量相等的最长子数组** → **映射 trick** `0 → -1, 1 → +1` 让问题变成 "**和 = 0 的最长子数组**" ⇔ 两 prefix sum 相等. hash 存首次 index, 求最大距离. `idx[0] = -1` 哨兵让从头开始命中时长度 = `i + 1`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 二进制数组 `nums` (只含 0 和 1). 求**含相等数量 0 和 1** 的**最长连续子数组** 的长度.

- 例: `[0, 1]` → 2, `[0, 1, 0]` → 2 (`[0, 1]`), `[0, 0, 1, 0, 0, 0, 1, 1]` → 6 (`[0, 1, 0, 0, 1, 1]`).

**中文**: 0 和 1 数量相等的最长子数组长度.

## Key Insights

1. **🔑 灵魂 mapping trick: `0 → -1, 1 → +1` / The killer mapping**

    "0 和 1 数量相等" 的子数组不好直接判. **换个视角**:

    - 把每个 **0 当作 -1**, **1 当作 +1**.
    - **子数组 0 和 1 数量相等** ⇔ **映射后和 = 0** (每个 -1 和 +1 相互抵消).
    - 再用 prefix sum: **子数组和 = 0** ⇔ **两 prefix sum 相等**.

    → 问题化简成: **"找相等 prefix 的最大 index 差"** — 就是前缀和 + hash 的通用模板.

    > **"约束条件不好数 → 换个数值编码"** 是**极巧妙的重塑** trick. 一个 mapping 让复杂问题变成家族老套路.

2. **🔑 hash 存"首次 index" — 求最长距离 / First-index map for longest distance**

    ```
    idx[sum] = 首次出现该 sum 的 index
    每次 curr_sum 命中 idx: dist = i - idx[curr_sum], 更新 maxLen
    不命中: idx[curr_sum] = i (保留首次)
    ```

    - **不覆盖已存**: 保留最早的 index → 未来任何命中都是**最大距离**.

    > 同 [0523 Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) 的存首次 index 思路. **"求最长/最远 → 存 endpoint 最早的"** 是求距离类通用 pattern.

3. **🔑 `idx[0] = -1` 哨兵 — 巧妙的 -1 / Sentinel: -1**

    为啥用 **-1** 而非 0?

    - 若从下标 0 到 i 的整段子数组和为 0, 我们想返回**长度 i + 1**.
    - `i - idx[0]` 需要等于 `i + 1` → `idx[0] = -1` ✓.
    - 若用 0, `i - 0 = i` 会**少 1**.

    ```
    例: nums = [0, 1], mapped = [-1, 1]
    i=0, sum=-1, 不在 idx, 记 idx[-1] = 0
    i=1, sum=0, 命中 idx[0]=-1, dist = 1 - (-1) = 2 ✓
    ```

    > **"哨兵 index = -1"** 是求"从头开始子数组" 的通用招式. 记牢这个 subtle 数字选择.

4. **🔑 前缀和家族的第四种 hash 用法 / Fourth pattern**

    | 题 | 求 | 存 | 判定 |
    |---|---|---|---|
    | [0560 Subarray Sum == K](../0560-subarray-sum-equals-k/README.md) | 个数 | count(sum) | `res += cnt[sum − k]` |
    | [0523 Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) | 存在, 长 ≥ 2 (mod k=0) | **first index**(mod) | `boundary − stored ≥ 2` |
    | [0974 Sums Divisible by K](../0974-subarray-sums-divisible-by-k/README.md) | 个数 (mod k=0) | count(mod) | `res += cnt[mod]` |
    | **0525 (本题)** | **最长 (sum=0)** | **first index**(sum) | **`max(i − stored)`** |

    → **前缀和 + hash** 家族 4 种 pattern: {个数, 存在, 最长} × {原和, mod}. **不同"目标 × 变换" 组合出不同 hash 语义**.

    > 记熟这 4 种, 遇到"子数组 + 某种和条件" 类题就能秒选正确 hash 结构.

5. **🔑 化简后完全是"最长 sum = 0 子数组" 通用模板 / Reduces to canonical LSS0**

    Yang 的循环内 `sum += (nums[i] == 1) ? 1 : -1` 就是**在线映射**. 之后:

    ```cpp
    if (idx.count(sum)) maxLen = max(maxLen, i - idx[sum]);
    else idx[sum] = i;
    ```

    → **模板化**. 若题目直接给 `nums = [+1, -1, ...]` 让求"最长和 0 子数组", 代码几乎一字不差.

    > **"变体识别 → 化简到通用问题"** 是解题的高段位思维. 本题的 mapping 就是化简的最后一步.

6. **🔑 复杂度 O(n) 时间, O(n) 空间 / Linear**

    - Time: 一遍扫, hash O(1).
    - Space: 最坏 O(n) 个不同 sum.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findMaxLength(vector<int>& nums) {
            unordered_map<int, int> idx;
            int maxLen = 0, sum = 0;
            idx[0] = -1;                                             // 哨兵: 让从 0 开始的整段 dist = i + 1

            for (int i = 0; i < (int)nums.size(); i++) {
                sum += (nums[i] == 1) ? 1 : -1;                      // 映射: 0 → -1, 1 → +1

                auto it = idx.find(sum);
                if (it != idx.end()) {
                    maxLen = max(maxLen, i - it->second);            // 更新最长
                } else {
                    idx[sum] = i;                                    // 只存首次
                }
            }
            return maxLen;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findMaxLength(self, nums: list[int]) -> int:
            idx = {0: -1}       # 哨兵
            max_len = s = 0
            for i, x in enumerate(nums):
                s += 1 if x == 1 else -1
                if s in idx:
                    max_len = max(max_len, i - idx[s])
                else:
                    idx[s] = i
            return max_len
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var findMaxLength = function(nums) {
        const idx = new Map();
        idx.set(0, -1);         // 哨兵
        let maxLen = 0, sum = 0;
        for (let i = 0; i < nums.length; i++) {
            sum += nums[i] === 1 ? 1 : -1;
            if (idx.has(sum)) {
                maxLen = Math.max(maxLen, i - idx.get(sum));
            } else {
                idx.set(sum, i);
            }
        }
        return maxLen;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(n) — hash map.

## 相关题目

- [0560. Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) — count 版
- [0523. Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) — first index + mod, 存在版
- [0974. Subarray Sums Divisible by K](../0974-subarray-sums-divisible-by-k/README.md) — count of mod 版
- [0001. Two Sum](../0001-two-sum/README.md) — "compensating value" 母题
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和母题
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- [0238. Product of Array Except Self](../0238-product-of-array-except-self/README.md) — 前缀积
- 0325\. Maximum Size Subarray Sum Equals k (待补) — 求"最长" 和 == k
- 1124\. Longest Well-Performing Interval (待补) — 同款 mapping trick + prefix
- 1546\. Maximum Number of Non-Overlapping Subarrays With Sum Equals Target (待补) — 前缀 + 贪心

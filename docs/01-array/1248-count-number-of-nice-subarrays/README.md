# 1248. Count Number of Nice Subarrays / 统计「优美子数组」

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Prefix Sum, Hash Map, Mapping Trick · 前缀和, 哈希表, 映射技巧
    - **Link**: [LeetCode](https://leetcode.com/problems/count-number-of-nice-subarrays/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count subarrays with exactly k odd numbers** → **mapping trick**: `odd → 1, even → 0`. Now the problem is **"count subarrays with sum == k"** — plug into [0560](../0560-subarray-sum-equals-k/README.md) template: prefix + hash count, `res += cnt[sum − k]`.
>
> **中文**: **恰含 k 个奇数的子数组个数** → **映射** `奇 → 1, 偶 → 0`, 问题变成 **"和 == k 的子数组个数"** — 直接套 0560 模板.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums` + `k`. 求**恰好含 k 个奇数** 的**连续子数组** 个数.

- 例: `nums=[1,1,2,1,1], k=3` → 2 (`[1,1,2,1]`, `[1,2,1,1]`).

**中文**: 恰含 k 个奇数的子数组个数.

## Key Insights

1. **🔑 灵魂 mapping trick: `奇 → 1, 偶 → 0` / Killer mapping**

    "恰好 k 个奇数" 不好直接数. **换个数值编码**:

    - **奇数 → 1**, **偶数 → 0**.
    - **子数组含 k 个奇数** ⇔ **映射后和 == k** — 每个奇贡献 1, 偶贡献 0.
    - 问题变成 [0560 Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) 一模一样.

    → **直接套模板**: prefix + hash count, `res += cnt[sum − k]`, seed `cnt[0] = 1`.

    > **"约束不好数 → 换编码" + "化简到通用模板"** 组合拳. 跟 [0525](../0525-contiguous-array/README.md) `0→-1, 1→+1` 同源思想 — **一个 mapping 决定归到哪个模板**.

2. **🔑 完全套 0560, 只换 sum 累加规则 / 1-line diff from 0560**

    Yang 的循环里只有**一行不同** 于 0560:

    ```cpp
    // 0560:  sum += x;
    // 1248:  sum += (x % 2 == 0) ? 0 : 1;      ← 只有这一行
    ```

    其他**完全一致**: `cnt[0] = 1`, 先查 `cnt[sum-k]` 加 res, 再 `cnt[sum]++`.

    > **模板题的美感**: 识别 pattern → 换一行 → 秒解. 这就是"模板熟练度"的战斗力.

3. **🔑 位运算优化 `x & 1` / Bitwise optimization**

    `x % 2` 可以写成 `x & 1` — 取最低位判奇偶. **常数快一点**, 面试提可加分:

    ```cpp
    sum += (x & 1);      // 直接加最低位 (奇 → 1, 偶 → 0)
    ```

    比 Yang 的三目 `(x % 2 == 0) ? 0 : 1` **更短更快**. **等价语义**, 代码短一半.

    > **`& 1` 判奇偶** 是 C 系老招. Python 也能 `x & 1` 但可读性一般, `x % 2` 更 Pythonic.

4. **🔑 备选思路: 三指针滑窗 O(n) O(1) / Sliding window alternative**

    另一种做法: 数"恰含 k 个" = "**至多 k 个** − **至多 k-1 个**".

    - `atMost(k)` = 用滑窗数**含 ≤ k 个奇数** 的子数组数.
    - 答案 = `atMost(k) − atMost(k−1)`.

    O(n) 时间, **O(1) 空间** (不用 hash). 稍难写但内存更省.

    > **"恰好 k = 至多 k − 至多 k-1"** 是滑窗计数的通用招. 见 0992, 0930 等.

5. **🔑 前缀和 + hash 家族最完美的"映射变体" / Cleanest mapping variant**

    对比家族已有:

    | 题 | 映射 | 归约到 |
    |---|---|---|
    | [0525](../0525-contiguous-array/README.md) | 0→-1, 1→+1 | 最长和 0 子数组 |
    | **1248 (本题)** | **奇→1, 偶→0** | **[0560](../0560-subarray-sum-equals-k/README.md) 个数** |
    | (可推广) 元音字母计数 | 元音→1, 其他→0 | 0560 或 0525 变体 |

    > **"约束条件 → 数值编码 → 归到基础模板"** 是极通用的问题化简范式. 记熟这套思维.

6. **🔑 复杂度 O(n) 时间, O(n) 空间 / Linear**

    - Time: 一遍扫, hash O(1).
    - Space: hash 最多 n+1 项 (sum 从 0 到 n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numberOfSubarrays(vector<int>& nums, int k) {
            unordered_map<int, int> cnt;
            cnt[0] = 1;                                              // 哨兵
            int res = 0, sum = 0;
            for (int x : nums) {
                sum += (x & 1);                                      // 映射 + 累加, 一步 (奇→1, 偶→0)
                auto it = cnt.find(sum - k);
                if (it != cnt.end()) res += it->second;              // 先查
                cnt[sum]++;                                          // 后计
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def numberOfSubarrays(self, nums: list[int], k: int) -> int:
            cnt = defaultdict(int)
            cnt[0] = 1
            s = res = 0
            for x in nums:
                s += x & 1                  # 奇→1, 偶→0
                res += cnt[s - k]           # defaultdict 缺 key 自动 0
                cnt[s] += 1
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var numberOfSubarrays = function(nums, k) {
        const cnt = new Map();
        cnt.set(0, 1);
        let sum = 0, res = 0;
        for (const x of nums) {
            sum += x & 1;
            res += cnt.get(sum - k) || 0;
            cnt.set(sum, (cnt.get(sum) || 0) + 1);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(n) — hash map.

## 相关题目

- [0560. Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) — **直接归约到此模板**
- [0525. Contiguous Array](../0525-contiguous-array/README.md) — 同款 mapping trick (0→-1)
- [0523. Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) — first index + mod
- [0974. Subarray Sums Divisible by K](../0974-subarray-sums-divisible-by-k/README.md) — count of mod
- [0001. Two Sum](../0001-two-sum/README.md) — compensating value 母题
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和母题
- 0930\. Binary Subarrays With Sum (待补) — 二进制 + 恰含 k 个 1, 同款
- 0992\. Subarrays with K Different Integers (待补) — "恰好 k = 至多 k − 至多 k-1" 滑窗
- 2588\. Count the Number of Beautiful Subarrays (待补) — XOR 前缀 + hash
- 1371\. Find the Longest Substring Containing Vowels in Even Counts (待补) — 元音位掩码 + prefix + hash

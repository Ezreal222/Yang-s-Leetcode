# 0560. Subarray Sum Equals K / 和为 K 的子数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Prefix Sum, Hash Map, Array · 前缀和, 哈希表, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/subarray-sum-equals-k/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count subarrays with sum == k** → **prefix sum + hash map**: `sum(i..j) == k ⇔ prefix[j+1] − prefix[i] == k ⇔ prefix[i] == curr − k`. Walk once, count how many previous prefixes equal `curr − k`. Seed `cnt[0] = 1` for subarrays starting at index 0.
>
> **中文**: **数和为 k 的子数组** → **前缀和 + hash map**: `子数组和 = 前缀差`. 一遍扫, 每步查"之前有几个 prefix 等于 curr - k". 初始 `cnt[0] = 1` 处理"从头开始" 的子数组.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums` + 整数 `k`. 求**和 == k** 的**连续子数组** 个数.

- 例: `nums = [1, 1, 1], k = 2` → 2 (`[1,1]` 前两个 + `[1,1]` 后两个).

**中文**: 求和为 k 的连续子数组个数.

## Key Insights

1. **🔑 朴素 O(n²) → 前缀和 + hash O(n) / Naive → prefix + hash**

    朴素: 双 for 枚举 (i, j), 累加 sum, 判 == k. O(n²). n = 20000 时 4×10⁸, TLE 边缘.

    **灵魂洞察**:

    ```
    sum(i..j) = prefix[j+1] - prefix[i]
    sum(i..j) == k
    ⇔ prefix[j+1] - prefix[i] == k
    ⇔ prefix[i] == prefix[j+1] - k       ← 关键
    ```

    → 遍历 j, 对每个 `curr = prefix[j+1]`, **查之前有多少个 prefix 值等于 `curr - k`** — hash map 计数 O(1).

2. **🔑 灵魂 hash map: `prefix 值 → 出现次数` / Hash: prefix-value → count**

    ```
    cnt[0] = 1                          // 哨兵: 空前缀 (对应"从下标 0 开始" 的子数组)
    sum = 0
    for x in nums:
        sum += x                        // 现在 sum = prefix[i+1]
        res += cnt[sum - k]             // 之前多少 prefix 等于 sum - k
        cnt[sum]++                      // 记账当前 prefix
    ```

    - **`sum - k`**: 我们要"找到"的历史 prefix 值.
    - **加 `cnt[sum - k]`** 一次 res 加它出现的次数 (可能多次 → 每次对应一个不同的子数组).

    > 跟 [0001 Two Sum](../0001-two-sum/README.md) 是**完全同款 "compensating value" 模式**: 0001 求 "另一个数 = target - x", 本题求 "另一个 prefix = curr - k".

3. **🔑 灵魂哨兵: `cnt[0] = 1` / Sentinel: cnt[0] = 1**

    为啥要预先 `cnt[0] = 1`?

    - 想数**"从下标 0 开始"** 的子数组 (即 `prefix[0] = 0`).
    - 例: nums = [3, 4], k = 7. sum = 3 (查 -4 → 无), sum = 7 (查 0 → **cnt[0] = 1** 命中!) → res = 1 ✓
    - 若不设, res = 0. 漏.

    > **"空前缀 = 0 存在 1 次"** 是这题的关键 setup. 少一行就 WA.

4. **🔑 先查再更新 / Query before update**

    ```
    res += cnt[sum - k]        // 先查
    cnt[sum]++                 // 后更新
    ```

    - 若**先更新**: `k = 0` 时会**把自己算成子数组** (从 i+1 到 i, 长度 0 的空子数组).
    - **先查后更**: 保证 hash 里只有**过去的** prefix, 不含当前.

    > 顺序细节, 一旦反了 `k = 0` case 立刻挂. 顺序敏感在 [0001 Two Sum](../0001-two-sum/README.md) 也见过 (先查再插).

5. **🔑 `long long` 防溢出 (防御性) / long long defensive**

    LC 数据 nums[i] ∈ [-1000, 1000], n ≤ 20000 → sum 最坏 ±2×10⁷, 不超 int. 但 Yang 用 long long **防御**: 若数据放宽到 nums[i] ∈ [-10⁵, 10⁵], sum 可到 2×10⁹ 超 int.

    > **防御性 cast 是老手 pattern**. 数值题第一反应查溢出.

6. **🔑 为啥 `sum - k` 不是 `k - sum`? / Why sum - k, not k - sum**

    从公式 `prefix[i] == curr - k` 反推. 想清楚"要找的是过去的 prefix, 该 prefix 满足什么条件"就懂了. 若写反 `k - curr`, 语义变成"未来 prefix 等于 k - curr", 但未来还没算, 逻辑错.

7. **🔑 跟前缀和家族的关系 / vs prefix-sum family**

    | 题 | 用途 | 关键 |
    |---|---|---|
    | [0303 Range Sum Query](../0303-range-sum-query-immutable/README.md) | 区间求和查询 | prefix 数组预处理 |
    | [0304 Range Sum 2D](../0304-range-sum-query-2d-immutable/README.md) | 二维区间求和 | 2D prefix |
    | [0238 Product Except Self](../0238-product-of-array-except-self/README.md) | 除自身外积 | 左积 × 右积 |
    | **0560 (本题)** | **和 == k 个数** | **prefix + hash 计数** |
    | 0974 Subarray Sums Divisible by K (待补) | 和 mod K == 0 个数 | prefix mod + hash |
    | 0525 Contiguous Array (待补) | 0 和 1 相等的最长子数组 | 换 mapping + hash |

    > **前缀和 + hash** 是**"子数组和/差/mod 特定值"** 一族的通用武器. 记熟.

8. **🔑 复杂度 O(n) 时间, O(n) 空间 / Linear**

    - Time: 一遍扫, hash O(1) 平均.
    - Space: O(n) 最坏 (所有 prefix 不同).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int subarraySum(vector<int>& nums, int k) {
            unordered_map<long long, int> cnt;
            cnt[0] = 1;                                              // 哨兵: 空前缀
            long long sum = 0;
            int res = 0;
            for (int x : nums) {
                sum += x;                                            // 当前 prefix
                auto it = cnt.find(sum - k);
                if (it != cnt.end()) res += it->second;              // 之前多少 prefix = sum - k
                ++cnt[sum];                                          // 记账 (先查后更)
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def subarraySum(self, nums: list[int], k: int) -> int:
            # defaultdict(int) 缺 key 自动返 0 — 免 if 判 (跟 C++ unordered_map operator[] 类似, 但更安全)
            cnt = defaultdict(int)
            cnt[0] = 1
            s = res = 0
            for x in nums:
                s += x
                res += cnt[s - k]       # 直接读, defaultdict 无 key 时是 0
                cnt[s] += 1              # 记账
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var subarraySum = function(nums, k) {
        const cnt = new Map();
        cnt.set(0, 1);
        let sum = 0, res = 0;
        for (const x of nums) {
            sum += x;
            // Map.get 缺 key 返 undefined, || 0 兜底
            res += cnt.get(sum - k) || 0;
            cnt.set(sum, (cnt.get(sum) || 0) + 1);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫, hash O(1) 平均.
- **Space**: O(n) — hash map.

## 相关题目

- [0001. Two Sum](../0001-two-sum/README.md) — "compensating value" 母题
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和母题
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- [0238. Product of Array Except Self](../0238-product-of-array-except-self/README.md) — 前缀积
- [0454. 4Sum II](../../03-hash-table/0454-4sum-ii/README.md) — 分组哈希 (同源思维)
- 0974\. Subarray Sums Divisible by K (待补) — prefix mod + hash
- [0523. Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) — prefix mod 判倍数 (存首次 boundary)
- 0525\. Contiguous Array (待补) — 换 mapping (0→-1) + prefix + hash 最长子数组
- 0325\. Maximum Size Subarray Sum Equals k (待补) — 求"最长" 版
- 1074\. Number of Submatrices That Sum to Target (待补) — 二维版, 压行 + 本题

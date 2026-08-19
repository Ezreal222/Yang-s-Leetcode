# 0523. Continuous Subarray Sum / 连续的子数组和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Prefix Sum, Hash Map, Math (Mod) · 前缀和, 哈希表, 模运算
    - **Link**: [LeetCode](https://leetcode.com/problems/continuous-subarray-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Does any subarray of length ≥ 2 have sum divisible by k?** → **prefix sum mod k + hash map of first-seen boundary**. `sumB − sumA divisible by k ⇔ sumB % k == sumA % k`. Store **first** boundary for each mod; on collision, check `curr_boundary − stored ≥ 2`.
>
> **中文**: **是否存在长 ≥ 2 的子数组, 和是 k 的倍数** → **前缀和 mod k + 首次出现 boundary 的 hash**. `两 prefix mod 相等 ⇔ 之间的子数组和被 k 整除`. 同 mod 命中且间距 ≥ 2 即真.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给数组 `nums` 和 `k`. 判断是否存在**长度 ≥ 2** 的**连续子数组**, 使其和是 `k` 的**倍数** (含 0 倍).

- 例: `nums=[23,2,4,6,7], k=6` → true (子数组 `[2, 4]` 和 6, 是 6×1).
- 例: `nums=[23,2,6,4,7], k=6` → true (整段 42, 是 6×7).

**中文**: 判长 ≥ 2 且和为 k 倍数的子数组是否存在.

## Key Insights

1. **🔑 灵魂化简: 两 prefix mod 相等 ⇒ 之间子数组和被 k 整除 / Same mod ⇒ divisible sum**

    **数论事实**:

    ```
    子数组 sum(i..j) = prefix[j+1] - prefix[i]
    sum(i..j) 被 k 整除
    ⇔ (prefix[j+1] - prefix[i]) % k == 0
    ⇔ prefix[j+1] % k == prefix[i] % k
    ```

    → 遍历前缀, **hash 存 (mod → 首次 boundary)**. 若当前 mod 之前出现过, 中间就是**整除子数组**.

    > **"差 mod = 差是 k 倍数"** 是模运算基础. 记牢: prefix sum + mod + hash 是"和整除" 类题的**通用武器**.

2. **🔑 hash 存**"首次 boundary"** 而非 count / Store first boundary, not count**

    跟 [0560 Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) 存 count 不同:

    - **0560 求个数** → 记 count, 每次 res += cnt[...].
    - **0523 求存在** (且要长度 ≥ 2) → 记**最早出现的 boundary**, 让距离最大化.

    ```
    if mod 已存在: 检查 boundary - stored >= 2 → 命中就返 true
    else: firstIdx[mod] = boundary
    ```

    > **"是否存在 + 需要距离 → 存首次索引"**. **"个数" → 存计数**. 两个语义, 两个 hash 用法.

3. **🔑 为啥**不能覆盖已有**? / Why never overwrite**

    若同 mod 已存, **不能** 更新为更晚的 boundary — 那样会缩短距离, 可能让"本来长 ≥ 2 的" 变成 < 2, 漏答案.

    ```
    保留最早的 stored → 未来任何 boundary 减它都是最大可能距离
    ```

    > **"想让 dist 最大 → 存 endpoint 之一的最早"**. 是求距离/长度类的通用 pattern.

4. **🔑 `firstIdx[0] = 0` 哨兵 / Sentinel: firstIdx[0] = 0**

    - 语义: **空前缀** 位于 boundary 0, mod 值 0.
    - 若某 boundary 后的 sum mod = 0 (即前几个数之和是 k 倍数), 就跟哨兵匹配 → 直接判长度 ≥ 2.
    - 例: `nums=[3,3], k=6` → 前两个数和 6, boundary 2 时 mod=0, 2-0=2 ≥ 2 → 真.

    > **哨兵覆盖"从头开始"** 的子数组. 跟 [0560](../0560-subarray-sum-equals-k/README.md) 的 `cnt[0] = 1` 同源.

5. **🔑 boundary = `i + 1` (消耗完 nums[i] 之后) / boundary = i + 1**

    Yang 用 **`boundary = i + 1`** 而非 i, 让**子数组长度 = boundary1 - boundary2** 直接对应.

    ```
    子数组 nums[a..b] = prefix[b+1] - prefix[a]
    长度 = (b+1) - a = boundary_after_b - boundary_before_a
    ```

    → `boundary - stored >= 2` 直接对应"长度 ≥ 2".

    > **boundary vs index 别混**. boundary 在**元素之间**, index 是**元素本身**. 前缀和习惯用 boundary.

6. **🔑 负数守卫: `if (sum < 0) sum += k` / Negative-mod guard**

    C++ `%` 对负数返负数 (与被除数同号). `(sum + nums[i]) % k` 若中间 sum 为负, 结果为负 → 跟正 mod 无法匹配, 逻辑错.

    Yang 加 `if (sum < 0) sum += k;` 强制正化. **本题 nums 全非负**, 这行严格不需要, 但**防御性写法** 好.

    > 数值题**先想 mod 语义**. C-语系 `%` 对负数有坑, Python `%` 不用担心 (数学定义, 返正).

7. **🔑 跟其他 prefix + hash 家族关系 / vs family**

    | 题 | 求 | 存 | 判定 |
    |---|---|---|---|
    | [0560 Subarray Sum == K](../0560-subarray-sum-equals-k/README.md) | 个数 | count | `cnt[sum - k]` 累加 |
    | **0523 (本题)** | **存在, 长 ≥ 2** | **首次 boundary** | **`boundary - stored ≥ 2`** |
    | 0974 Divisible by K (待补) | 个数 | count(mod) | `cnt[mod]` 累加 |
    | 0525 Contiguous Array (待补) | 最长 | 首次 boundary | max(dist) |

    > **"个数 → count map, 距离/长度 → 首次 index map"**. 记熟这条区分.

8. **🔑 复杂度 O(n) 时间, O(min(n, k)) 空间 / Linear**

    - Time: 一遍扫, hash O(1).
    - Space: mod 值只有 k 种 → hash 最多 min(n, k) 项.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool checkSubarraySum(vector<int>& nums, int k) {
            unordered_map<int, int> firstIdx;
            firstIdx[0] = 0;                                         // 哨兵: 空前缀 boundary 0
            int sum = 0;
            for (int i = 0; i < (int)nums.size(); ++i) {
                sum = (sum + nums[i]) % k;
                if (sum < 0) sum += k;                               // 负数守卫 (防御性)
                int boundary = i + 1;                                // 消耗 nums[i] 后的边界
                auto it = firstIdx.find(sum);
                if (it != firstIdx.end()) {
                    if (boundary - it->second >= 2) return true;     // 长度 ≥ 2
                    // 已存在 → 保留最早的 (不覆盖)
                } else {
                    firstIdx[sum] = boundary;
                }
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def checkSubarraySum(self, nums: list[int], k: int) -> bool:
            first_idx = {0: 0}      # 哨兵
            s = 0
            for i, x in enumerate(nums):
                s = (s + x) % k     # Python % 对负数返正 (数学定义) — 天然安全
                boundary = i + 1
                if s in first_idx:
                    if boundary - first_idx[s] >= 2:
                        return True
                    # 已存在, 不覆盖 (保留最早)
                else:
                    first_idx[s] = boundary
            return False
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {boolean}
     */
    var checkSubarraySum = function(nums, k) {
        const firstIdx = new Map();
        firstIdx.set(0, 0);
        let sum = 0;
        for (let i = 0; i < nums.length; i++) {
            sum = (sum + nums[i]) % k;
            // JS % 对负数返负 (跟 C++ 同), 严格需要 +k 兜; 本题 nums 全非负故省
            if (sum < 0) sum += k;
            const boundary = i + 1;
            if (firstIdx.has(sum)) {
                if (boundary - firstIdx.get(sum) >= 2) return true;
            } else {
                firstIdx.set(sum, boundary);
            }
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(min(n, k)) — hash map 最多 k 种 mod 值.

## 相关题目

- [0560. Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) — 前缀和 + hash 个数
- [0001. Two Sum](../0001-two-sum/README.md) — "compensating value" 母题
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和母题
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- [0238. Product of Array Except Self](../0238-product-of-array-except-self/README.md) — 前缀积
- 0974\. Subarray Sums Divisible by K (待补) — 数被 k 整除的子数组**个数** (换 count map)
- 0525\. Contiguous Array (待补) — 0 和 1 相等的**最长** 子数组 (mapping trick)
- 0325\. Maximum Size Subarray Sum Equals k (待补) — 求"最长" 和 == k
- 1590\. Make Sum Divisible by P (待补) — 求"最短" 删除后被整除

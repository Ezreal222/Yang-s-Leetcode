# 0974. Subarray Sums Divisible by K / 和可被 K 整除的子数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Prefix Sum, Hash Map, Math (Mod) · 前缀和, 哈希表, 模运算
    - **Link**: [LeetCode](https://leetcode.com/problems/subarray-sums-divisible-by-k/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count subarrays whose sum is divisible by k** → **prefix mod k + hash map of counts**. `sum(i..j) % k == 0 ⇔ prefix[j+1] % k == prefix[i] % k`. Seed `cnt[0] = 1`; walk once, `res += cnt[curr_mod]` before incrementing. Use `((x % k) + k) % k` for negative safety.
>
> **中文**: **数和被 k 整除的子数组个数** → **前缀 mod + hash count**. `子数组和被 k 整除 ⇔ 两 prefix mod 相等`. 先查再计. `((x % k) + k) % k` 处理负数.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给数组 `nums` 和整数 `k`. 求**和被 k 整除** 的**非空连续子数组** 的**个数**.

- 例: `nums=[4,5,0,-2,-3,1], k=5` → 7. (`[4,5,0,-2,-3,1]`, `[5]`, `[5,0]`, `[5,0,-2,-3]`, `[0]`, `[0,-2,-3]`, `[-2,-3]`)

**中文**: 数被 k 整除的子数组个数.

## Key Insights

1. **🔑 灵魂同 [0523](../0523-continuous-subarray-sum/README.md): 两 prefix mod 相等 ⇒ 差被 k 整除 / Same identity**

    ```
    sum(i..j) 被 k 整除
    ⇔ prefix[j+1] % k == prefix[i] % k
    ```

    → 遍历 curr, 对每个 curr_mod, **之前有多少 prefix 跟它同 mod, 就多少个整除子数组**.

    > **数论识别**: 两个数**同余** ⇔ **差被 k 整除**. prefix + mod 是"和被 k 整除" 类的**必反射**.

2. **🔑 本题存 count (求个数) — 跟 0523 家族对比 / Store count, not first boundary**

    | 题 | 求 | 存 | 判定 |
    |---|---|---|---|
    | [0560 Subarray Sum == K](../0560-subarray-sum-equals-k/README.md) | 个数 (和等于 k) | count | `res += cnt[sum − k]` |
    | [0523 Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) | 存在 + 长 ≥ 2 (被 k 整除) | **首次 boundary** | `boundary − stored ≥ 2` |
    | **0974 (本题)** | **个数 (被 k 整除)** | **count** | **`res += cnt[curr_mod]`** |

    → **"个数 → count map, 存在/距离 → first index map"**. 一族三题, 三种 hash 用法.

3. **🔑 `((x % k) + k) % k` — 负数安全的 mod / Safe negative mod**

    Yang 的一行 `((sum + x) % k + k) % k` 是**C++ 标准防御**:

    - C++ `%` 对负数返负 (与被除数同号): `-3 % 5 = -3`.
    - `(-3 + 5) % 5 = 2` ✓ (数学 mod).
    - **两次 mod** 保证结果在 `[0, k-1]`.

    > 本题**nums 可含负数**, 这行**严格必需** (跟 0523 只是防御性不同). Python `%` 天然正, 不用.

4. **🔑 先查后计 / Query before increment**

    ```
    res += cnt[curr_mod]     // 先查: 之前多少同 mod
    cnt[curr_mod]++          // 后计: 记账本次
    ```

    - 若**先计**: 会把当前 prefix 自身也算进去 → 多一个"空子数组" 计数.
    - **先查后计**保证只跟**过去** 的 prefix 匹配.

    > 同 [0560](../0560-subarray-sum-equals-k/README.md) 的先查后插原则 — 顺序敏感.

5. **🔑 `cnt[0] = 1` 哨兵 / Sentinel**

    - 处理"从下标 0 开始被整除的子数组".
    - 例: `nums=[5], k=5` — sum=5, mod=0. 若无哨兵 `cnt[0]=0`, res=0 (错). 有哨兵 res=1 ✓.

    > 跟 0523 / 0560 同源. "空前缀 是 mod = 0" 是通用哨兵.

6. **🔑 组合数学: `cnt[m] 出现次数 c → 贡献 C(c, 2) 对 / Combinatorial insight**

    另一种看法: 遍历完后, 对每个 mod 值 m, 若 cnt[m] = c (含哨兵), 贡献 **C(c, 2) = c(c-1)/2** 对整除子数组.

    - 因为**c 个 prefix 中任选 2 个**都满足"差被 k 整除".

    ```cpp
    // 等价写法 (面试展示)
    for (auto& [_, c] : cnt) res += c * (c - 1) / 2;
    ```

    Yang 的"一遍扫累加" 是**在线** 版本; 上面是**最后统计** 版. **数学等价**.

    > **"计数配对" 类题很多能表达成组合数**. 这道题就是典型.

7. **🔑 复杂度 O(n) 时间, O(min(n, k)) 空间 / Linear**

    - Time: 一遍扫.
    - Space: mod 值最多 k 种.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int subarraysDivByK(vector<int>& nums, int k) {
            unordered_map<long long, int> cnt;
            cnt[0] = 1;                                              // 哨兵
            long long sum = 0;
            int res = 0;
            for (int x : nums) {
                sum = ((sum + x) % k + k) % k;                       // 负数安全 mod
                auto it = cnt.find(sum);
                if (it != cnt.end()) res += it->second;              // 先查
                ++cnt[sum];                                          // 后计
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def subarraysDivByK(self, nums: list[int], k: int) -> int:
            cnt = defaultdict(int)
            cnt[0] = 1
            s = res = 0
            for x in nums:
                s = (s + x) % k     # Python % 对负数返正 (数学定义) — 天然安全
                res += cnt[s]
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
    var subarraysDivByK = function(nums, k) {
        const cnt = new Map();
        cnt.set(0, 1);
        let sum = 0, res = 0;
        for (const x of nums) {
            // JS % 对负数返负 (跟 C++ 同), 必须 +k 兜正
            sum = ((sum + x) % k + k) % k;
            res += cnt.get(sum) || 0;
            cnt.set(sum, (cnt.get(sum) || 0) + 1);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(min(n, k)) — mod 值最多 k 种.

## 相关题目

- [0560. Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) — 和 == k 个数 (count map)
- [0523. Continuous Subarray Sum](../0523-continuous-subarray-sum/README.md) — 和被 k 整除 + 长 ≥ 2 存在 (first index map)
- [0001. Two Sum](../0001-two-sum/README.md) — "compensating value" 母题
- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀和母题
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- [0238. Product of Array Except Self](../0238-product-of-array-except-self/README.md) — 前缀积
- [0525. Contiguous Array](../0525-contiguous-array/README.md) — 0 和 1 相等的**最长** (mapping trick)
- 0325\. Maximum Size Subarray Sum Equals k (待补) — 求"最长"
- 1590\. Make Sum Divisible by P (待补) — 求"最短" 删除后被整除
- 1074\. Number of Submatrices That Sum to Target (待补) — 二维版

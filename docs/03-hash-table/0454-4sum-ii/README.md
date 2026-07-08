# 0454. 4Sum II / 四数相加 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Map, Meet in the Middle, Counting · 哈希表, 折半搜索, 计数
    - **Link**: [LeetCode](https://leetcode.com/problems/4sum-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count tuples across 4 arrays summing to 0** → **split 4 into 2 + 2**: hash all `a + b` sums (nums1 × nums2) with counts, then for each `c + d` (nums3 × nums4) add `map[-(c + d)]`. **O(n²)** time.
>
> **中文**: **四数组各选一, 求和 = 0 的组合数** → **拆两半**: 先哈希 nums1×nums2 所有 a+b 的次数, 再枚举 nums3×nums4 查 `-(c+d)` 的次数累加. **O(n²)** 时间.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 4 个等长数组 `nums1..4` (长 n). 求四元组 `(i, j, k, l)` 的**个数** 使 `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`.

**中文**: 四数组各选一个和为 0 的组合数.

## Key Insights

1. **🔑 朴素 O(n⁴) → 分组 O(n²) / Naive O(n⁴) → split-and-hash O(n²)**

    暴力四层 for 试所有组合: n = 200 时是 200⁴ = 1.6 × 10⁹, TLE.

    **哈希拆分**: 把 4 数组拆两半 (2 + 2). 每半 O(n²) 枚举, 一半"预存", 另一半"查表":

    ```
    for a in nums1:
        for b in nums2:
            map[a + b]++              # 预存: n² 个 sum 及其次数
    for c in nums3:
        for d in nums4:
            count += map[-(c + d)]    # 查表: n² 次 O(1) lookup
    ```

    - 时间: O(n²) + O(n²) = **O(n²)** — 从 10⁹ 降到 10⁴ 数量级.
    - 空间: O(n²) — map 最坏 n² 个 key.

    > **折半搜索 (Meet in the Middle)**: k 个东西**要枚举组合** → 拆两半, 一半预存一半查. 是**指数级 → 亚指数** 或 **多项式 → 亚多项式** 的通用技巧.

2. **🔑 为啥 2+2 而不是 1+3? / Why 2+2, not 1+3**

    | 拆法 | 预存 | 查表 | 总时间 |
    |---|---|---|---|
    | **2 + 2** (Yang) | O(n²) | O(n²) | **O(n²)** ✅ |
    | 1 + 3 | O(n) | O(n³) | O(n³) |
    | 3 + 1 | O(n³) | O(n) | O(n³) |

    **两半平衡** 时总时间最优 (类似"两个数相乘, 和固定时积最小 当两数相等" 的经典不等式).

    > **k 分两半的原理**: min(a + b, ab) s.t. a + b = k → 取 a = b = k/2. **6Sum II 拆 3+3, 8Sum II 拆 4+4**, 依此类推.

3. **🔑 哈希值是 count 不是 bool / Value is count, not just existence**

    问的是**组合数** 而不是"存在吗" → map 存 `(sum → 出现次数)`. 因为**同一个 sum** 可能由**多种 (a, b)** 组合达成, 都要计数.

    ```
    nums1 = [1, 2], nums2 = [-1, 0]
    a+b: (1)+(-1)=0, (1)+(0)=1, (2)+(-1)=1, (2)+(0)=2
    map = {0:1, 1:2, 2:1}
    ```

    > **"个数" 问题 map value 一律是 count**. 跟 [0001](../../01-array/0001-two-sum/README.md) (值 → index) 语义不同, 记住区分.

4. **🔑 `unordered_map::operator[]` 缺 key 时返 0 / operator[] auto-defaults**

    Yang 的 `count += mp[-(c + d)]` 一行没判"存不存在" — 因为 C++ `unordered_map<int, int>` 的 `operator[]` 在**缺 key 时**默认构造 `int()` = 0, 加 0 无害.

    > **小心副作用**: 这**会插入**一个 count=0 的条目 (虽然值是 0). 对本题**无影响** (答案对). 但**性能敏感** 时可能想 `.count()` 或 `.find()` 先判, 避免污染 map. 这题不管.

5. **🔑 跟 [0001](../../01-array/0001-two-sum/README.md) 的关系: "compensating value" 的推广 / Generalizing the compensator pattern**

    - 0001: 单数组, 固定当前 x, 查 `target - x`.
    - **本题**: 拆 4 → 2+2, 固定 `(c+d)`, 查 `-(c+d)` 在**另一半 sum 空间** 里的次数.

    > **同一 mental model**: 找"能配对的另一半", hash 把 O(n) 查降到 O(1). 只是维度从 1 升到 2.

6. **🔑 跟 0018 4Sum 的区别 / vs 0018 4Sum**

    | | 0454 (本题) | 0018 4Sum |
    |---|---|---|
    | 输入 | **4 个数组** | **1 个数组** |
    | 输出 | **组合数** | **所有具体四元组** (值, 去重) |
    | 方法 | 分组哈希 | **排序 + 双层 for + 对撞双指针 + 去重** |
    | 时间 | O(n²) | O(n³) |

    > 输入形态**决定** 用哪种技术. 4 个独立数组 → 分组哈希; 单数组 → 排序 + 双指针.

7. **复杂度 O(n²) 时间, O(n²) 空间 / Quadratic**

    - Time: 两个 O(n²) 循环相加.
    - Space: map 最坏 n² 个不同 sum.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int fourSumCount(vector<int>& nums1, vector<int>& nums2,
                         vector<int>& nums3, vector<int>& nums4) {
            unordered_map<int, int> mp;                              // (a + b) → count
            for (int a : nums1)
                for (int b : nums2)
                    mp[a + b]++;                                     // 预存 O(n²) 个和

            int count = 0;
            for (int c : nums3)
                for (int d : nums4)
                    count += mp[-(c + d)];                           // 查 -(c+d) 的次数
            return count;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def fourSumCount(self, nums1, nums2, nums3, nums4):
            # Counter 自带"缺 key 返 0" 的语义, 跟 C++ unordered_map<int,int> 一致
            # sum(a+b for ...) 用 Counter(...) 一步计数
            # 等价 C++ 两层 for + map[a+b]++, 但更 Pythonic
            ab = Counter(a + b for a in nums1 for b in nums2)
            # 对每对 (c, d), 累加 ab[-(c+d)] — Counter 缺 key 也返 0
            return sum(ab[-(c + d)] for c in nums3 for d in nums4)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums1
     * @param {number[]} nums2
     * @param {number[]} nums3
     * @param {number[]} nums4
     * @return {number}
     */
    var fourSumCount = function(nums1, nums2, nums3, nums4) {
        // Map 比 Object 好: 数字 key 保持类型, 且 .get() 缺 key 返 undefined 我们主动兜 0
        const mp = new Map();
        for (const a of nums1) {
            for (const b of nums2) {
                const s = a + b;
                mp.set(s, (mp.get(s) || 0) + 1);         // ?? 0 或 || 0: JS 惯用 default 兜底
            }
        }
        let count = 0;
        for (const c of nums3) {
            for (const d of nums4) {
                count += mp.get(-(c + d)) || 0;          // 缺 key 加 0
            }
        }
        return count;
    };
    ```

## Complexity

- **Time**: O(n²) — 两轮双层 for.
- **Space**: O(n²) — 最坏 n² 个不同 sum.

## 相关题目

- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — 一维版, "compensating value" 母题
- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组母题
- [0349. Intersection of Two Arrays](../0349-intersection-of-two-arrays/README.md) — hash set 交集
- [0202. Happy Number](../0202-happy-number/README.md) — hash set 判环
- 0167\. Two Sum II - Input Array Is Sorted (待补) — 排序数组 + 对撞双指针
- 0015\. 3Sum (待补) — 排序 + 对撞 + 去重
- 0018\. 4Sum (待补) — 单数组 4 元组, 排序 + 双层 for + 双指针
- [(§10 DP 折半搜索) topic-meet-in-the-middle](../../10-dp/topic-meet-in-the-middle.md) — 折半搜索的更多应用
- 2035\. Partition Array Into Two Arrays to Minimize Sum Difference (待补) — 折半 + 二分
- 1755\. Closest Subsequence Sum (待补) — 折半 + 排序 + 二分

# 0018. 4Sum / 四数之和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, Sort, Array, Dedupe · 双指针, 排序, 数组, 去重
    - **Link**: [LeetCode](https://leetcode.com/problems/4sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **All unique quadruplets summing to target** → **sort** + **2 outer loops (i, j)** + **inner opposite two pointers** + **4-level dedupe**; use **`long long`** to guard sum overflow.
>
> **中文**: **四元组和 = target, 去重** → **排序** + **外层双 for (i, j)** + **内层对撞双指针** + **4 层去重**; **`long long`** 防 sum 溢出.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums` + `target`. 找**所有**满足 `nums[a] + nums[b] + nums[c] + nums[d] == target` 的**四元组**. **答案不能重复**.

**中文**: 四数之和 = target 的所有不重复四元组.

## Key Insights

1. **🔑 [0015 3Sum](../0015-3sum/README.md) 的直接推广: 拆维 +1 / Direct extension of 0015**

    3Sum = 枚举 i × 剩余 2Sum. **4Sum = 枚举 (i, j) × 剩余 2Sum**. 每加一维, **多套一层 for + 多一层去重**.

    ```
    for i:                              (i 层去重)
        for j > i:                      (j 层去重)
            two-pointer 2Sum on nums[j+1..n-1] with target - nums[i] - nums[j]
            (left / right 层去重)
    ```

    > **k-Sum 的通用模板**: 外层 (k-2) 个 for, 内层双指针. 总 O(n^(k-1)).

2. **🔑 4 层去重的位置 / Dedupe at 4 levels**

    ```cpp
    // 层 1: i (跟 0015 同款)
    if (i > 0 && nums[i] == nums[i - 1]) continue;

    // 层 2: j — 注意起点是 j > i + 1 (**不是** j > 0)
    if (j > i + 1 && nums[j] == nums[j - 1]) continue;

    // 层 3, 4: left / right (命中后 skip)
    ```

    > **`j > i + 1` 而不是 `j > 0`** 是易错点. 语义: **j 是这一轮 i 的第一个** 时不判重, 否则会漏 `[1, 1, 2, 2]` 这种合法情况.

3. **🔑 `long long` 防溢出 / Overflow guard with `long long`**

    Yang 的关键防御:

    ```cpp
    long long sum = (long long)nums[i] + nums[j] + nums[left] + nums[right];
    ```

    - LC 允许 `nums[i]` 到 `1e9`, target 到 `1e9`.
    - 4 个 `int` 相加**可能超过 INT_MAX** (~2.1e9). 3Sum 也有这问题, 但**3 数**通常刚好在边界.
    - **`(long long)nums[i]` cast 触发**整个表达式提升为 long long → 后续加法都用 64 位, 无溢出.

    > **k ≥ 4 时**这一步几乎必须. 面试**主动提**防溢出的 cast 是加分动作.

4. **🔑 提前终止 —  4Sum 比 3Sum 复杂 / Early break for 4Sum: trickier than 3Sum**

    3Sum 里 `nums[i] > 0 → break` 直接. 4Sum 因为 **target 可正可负**, 不能这么简单.

    严格版剪枝 (Yang 未写, 也 OK):

    ```cpp
    // 若当前 nums[i] × 4 已 > target 且 nums[i] > 0 → 后续更大 → 无解
    if (nums[i] > target && nums[i] > 0) break;
    // 类似 j 层, 内层双指针也可加"最小和" 剪枝
    ```

    > **不加剪枝仍是 O(n³)**, LC 可过. 加了常数快. **面试问 "还能优化吗?"** 就搬这个.

5. **🔑 双指针的收缩逻辑跟 [3Sum](../0015-3sum/README.md) 一模一样 / Inner logic identical to 0015**

    ```
    while left < right:
        sum = i + j + left + right
        if sum == target: 收 + 4 层去重后段 + 双端推进
        elif sum < target: left++
        else: right--
    ```

    差别**只在维度**. 内核就是 3Sum 的对撞双指针.

6. **🔑 复杂度 O(n³) 时间, O(1) 额外 / Cubic time, constant extra**

    - Sort: O(n log n).
    - 三层 (i × j × 双指针): O(n³).
    - 输出不算.

    > **想更快?** 用 [0454 4Sum II](../../03-hash-table/0454-4sum-ii/README.md) 的**分组哈希**! 但 0454 是 **4 个数组各选一 + 只求个数**; 本题 **1 数组 + 求具体四元组**, 不能直接搬. 输入形态决定技术选型.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> fourSum(vector<int>& nums, int target) {
            vector<vector<int>> res;
            sort(nums.begin(), nums.end());
            int n = nums.size();
            for (int i = 0; i < n; i++) {
                if (i > 0 && nums[i] == nums[i - 1]) continue;                     // i 去重
                for (int j = i + 1; j < n; j++) {
                    if (j > i + 1 && nums[j] == nums[j - 1]) continue;             // j 去重
                    int left = j + 1, right = n - 1;
                    while (left < right) {
                        long long sum = (long long)nums[i] + nums[j]
                                      + nums[left] + nums[right];                  // 防溢出
                        if (sum == target) {
                            res.push_back({nums[i], nums[j], nums[left], nums[right]});
                            while (left < right && nums[left] == nums[left + 1]) left++;
                            while (left < right && nums[right] == nums[right - 1]) right--;
                            left++; right--;
                        } else if (sum < target) {
                            left++;
                        } else {
                            right--;
                        }
                    }
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def fourSum(self, nums: list[int], target: int) -> list[list[int]]:
            # Python int 是任意精度, 天然无溢出 — 不用 long long cast
            nums.sort()
            res: list[list[int]] = []
            n = len(nums)
            for i in range(n):
                if i > 0 and nums[i] == nums[i - 1]: continue
                for j in range(i + 1, n):
                    if j > i + 1 and nums[j] == nums[j - 1]: continue
                    left, right = j + 1, n - 1
                    while left < right:
                        s = nums[i] + nums[j] + nums[left] + nums[right]
                        if s == target:
                            res.append([nums[i], nums[j], nums[left], nums[right]])
                            while left < right and nums[left] == nums[left + 1]: left += 1
                            while left < right and nums[right] == nums[right - 1]: right -= 1
                            left += 1; right -= 1
                        elif s < target:
                            left += 1
                        else:
                            right -= 1
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number[][]}
     */
    var fourSum = function(nums, target) {
        // JS 数字是 double, 安全整数最大 2^53 — 4 个 int 相加不会溢出到不安全区
        // 无需类似 C++ 的 long long cast, 但要记住 JS Number 的这个特性
        nums.sort((a, b) => a - b);
        const res = [];
        const n = nums.length;
        for (let i = 0; i < n; i++) {
            if (i > 0 && nums[i] === nums[i - 1]) continue;
            for (let j = i + 1; j < n; j++) {
                if (j > i + 1 && nums[j] === nums[j - 1]) continue;
                let left = j + 1, right = n - 1;
                while (left < right) {
                    const sum = nums[i] + nums[j] + nums[left] + nums[right];
                    if (sum === target) {
                        res.push([nums[i], nums[j], nums[left], nums[right]]);
                        while (left < right && nums[left] === nums[left + 1]) left++;
                        while (left < right && nums[right] === nums[right - 1]) right--;
                        left++; right--;
                    } else if (sum < target) {
                        left++;
                    } else {
                        right--;
                    }
                }
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n³) — 排序 O(n log n) + 三层扫 O(n³).
- **Space**: O(1) 额外.

## 相关题目

- [0015. 3Sum](../0015-3sum/README.md) — 母题, 拆维套路
- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — 一维版, hash
- [0454. 4Sum II](../../03-hash-table/0454-4sum-ii/README.md) — 4 数组分组哈希, O(n²)
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针
- 0016\. 3Sum Closest (待补) — 最接近 target 的 3 数之和
- 0259\. 3Sum Smaller (待补) — 3 数之和 < target 计数
- 0167\. Two Sum II - Input Array Is Sorted (待补) — 有序 2Sum
- 0011\. Container With Most Water (待补) — 对撞 + 贪心

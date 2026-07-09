# 0015. 3Sum / 三数之和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, Sort, Array, Dedupe · 双指针, 排序, 数组, 去重
    - **Link**: [LeetCode](https://leetcode.com/problems/3sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **All unique triplets summing to 0** → **sort** first; outer `i`, inner **opposite-direction two pointers** `left/right` in `[i+1, n-1]`; **dedupe at all 3 levels**; early break when `nums[i] > 0`.
>
> **中文**: **三元组和 = 0, 去重** → **排序**, 外层 i, 内层**对撞双指针** left/right 在 `[i+1, n-1]`; **三层去重** (i / left / right); `nums[i] > 0` 提前终止.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums`. 找**所有**满足 `nums[i] + nums[j] + nums[k] == 0` 的**三元组** (i, j, k 互不相等). **答案不能重复**.

**中文**: 三数之和为 0 的所有不重复三元组.

## Key Insights

1. **🔑 核心 idea: 排序 + fix i + 剩余 2Sum / Sort + fix i + inner 2Sum**

    3Sum 拆成"**枚举 i × 剩余 2Sum**":

    ```
    for i in 0..n-1:
        target = -nums[i]
        find all pairs (left, right) in nums[i+1..n-1] with sum == target
    ```

    剩余 2Sum 若数组**有序** 就 O(n) — 对撞双指针. 于是总 O(n²).

    > **"降一维" 的通用套路**: k-Sum → (k-1)-Sum. 拿 3Sum 熟, 4Sum / kSum 就是套壳.


2. **🔑 为啥先排序 / Why sort first**

    有序数组给了对撞双指针"**方向感**":

    - 若 `sum < 0` → 需要**更大**的 → `left++`.
    - 若 `sum > 0` → 需要**更小**的 → `right--`.
    - 若 `sum == 0` → 命中.

    没有排序, 双指针**不知道该动哪个**. 排序 O(n log n) 是一次性成本, 换来内层 O(n).

    > **"看到不定长子数组 / 三元组 + 求和相关" → 排序 + 双指针** 是一族解法.

3. **🔑 三层去重 (三个"重复位置") / Dedupe at three levels**

    结果**不重复** 是本题的绊脚石. 三处都要 skip:

    ```cpp
    // 层 1: 外层 i
    if (i > 0 && nums[i] == nums[i - 1]) continue;

    // 找到一组后:
    // 层 2: left
    while (left < right && nums[left] == nums[left + 1]) left++;
    // 层 3: right
    while (left < right && nums[right] == nums[right - 1]) right--;
    left++; right--;
    ```

    - **i 去重**: 若 `nums[i] == nums[i-1]`, 内层扫过的空间跟上一轮一样, 产生同样的三元组.
    - **left/right 去重**: **命中之后** 才 skip — 因为要**先收一组**, 再跳过后续同值的.
    - **`i > 0`** 守卫: 第一次 i 不要跟 `nums[-1]` 比. Yang 的写法漂亮.

    > **"跳到最后一个相同值, 然后再 ++/--"** — 记住这个双 while 模板. 去重题的通用手法.

4. **🔑 提前终止: `nums[i] > 0` → break / Early break on positive**

    已排序 → `nums[i] > 0` 后**所有 nums[j], nums[k]** (j, k > i) 都 ≥ nums[i] > 0 → 三个正数**和不可能 = 0** → 直接 break, 后面全跳.

    > 常数级优化, 但**面试提这一句** 显示注意力.

5. **🔑 双指针的收缩逻辑 / Two-pointer squeeze**

    ```
    left = i + 1, right = n - 1
    while left < right:
        sum = nums[i] + nums[left] + nums[right]
        if sum == 0: 收结果 + 3 层去重后 双端推进
        elif sum < 0:  left++       # 需要更大
        else:          right--      # 需要更小
    ```

    每步**必收缩一边**, `left, right` 单调, 内层 O(n).

6. **🔑 hash 方案 vs 双指针 / Hash-set alternative**

    | 方法 | Time | Space | 去重 |
    |---|---|---|---|
    | **排序 + 双指针** (Yang) | O(n²) | O(1) 额外 (原地排序) | **模板化, 三层 while** |
    | 排序 + hash set | O(n²) | O(n) | 用 set of tuples 天然去重, 代码更短但慢 |

    > **面试首选排序双指针** — 内存少, 常数快, 通用性强. Hash 版是备胎.

7. **🔑 跟 [0001 Two Sum](../../01-array/0001-two-sum/README.md) 的关系 / Relation to 0001**

    | | 0001 Two Sum | **0015 3Sum** |
    |---|---|---|
    | 元数 | 2 | 3 |
    | 输入 | 无序 | **允许排序 (只要下标不用)** |
    | 数据结构 | hash map | **对撞双指针** |
    | 输出 | 一对下标 | **所有三元组值 (去重)** |
    | 去重 | 无 | **3 层** |

    > 0001 保留原下标 → 只能 hash. 0015 返值不返下标 → **允许排序** → 换更优的双指针.

8. **复杂度 O(n²) 时间, O(1) 额外空间 / Quadratic time, constant extra**

    - Sort: O(n log n).
    - 双层扫: O(n²).
    - 输出不算, 排序原地 (C++/Java) 或 log n 栈 (Python Timsort).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> threeSum(vector<int>& nums) {
            vector<vector<int>> res;
            sort(nums.begin(), nums.end());
            int n = nums.size();
            for (int i = 0; i < n; i++) {
                if (nums[i] > 0) break;                                  // 提前终止
                if (i > 0 && nums[i] == nums[i - 1]) continue;           // i 层去重

                int left = i + 1, right = n - 1;
                while (left < right) {
                    int sum = nums[i] + nums[left] + nums[right];
                    if (sum == 0) {
                        res.push_back({nums[i], nums[left], nums[right]});
                        while (left < right && nums[left] == nums[left + 1]) left++;   // left 去重
                        while (left < right && nums[right] == nums[right - 1]) right--; // right 去重
                        left++; right--;                                 // 双端推进
                    } else if (sum < 0) {
                        left++;                                          // 需要更大
                    } else {
                        right--;                                         // 需要更小
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
        def threeSum(self, nums: list[int]) -> list[list[int]]:
            # sorted() 返新列表, nums.sort() 原地. 这里用原地省一次拷贝 (跟 C++ 一致)
            nums.sort()
            res: list[list[int]] = []
            n = len(nums)
            for i in range(n):
                if nums[i] > 0: break
                if i > 0 and nums[i] == nums[i - 1]: continue
                left, right = i + 1, n - 1
                while left < right:
                    s = nums[i] + nums[left] + nums[right]
                    if s == 0:
                        res.append([nums[i], nums[left], nums[right]])
                        # while 跳过重复 — Python 语法同 C++
                        while left < right and nums[left] == nums[left + 1]: left += 1
                        while left < right and nums[right] == nums[right - 1]: right -= 1
                        left += 1; right -= 1
                    elif s < 0:
                        left += 1
                    else:
                        right -= 1
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number[][]}
     */
    var threeSum = function(nums) {
        // JS 默认 .sort() 是字典序 (会把 [10, 2] 排成 [10, 2]!). 数字要传比较函数
        // (a, b) => a - b 是升序数值排序, 跟 C++ std::sort 一致
        nums.sort((a, b) => a - b);
        const res = [];
        const n = nums.length;
        for (let i = 0; i < n; i++) {
            if (nums[i] > 0) break;
            if (i > 0 && nums[i] === nums[i - 1]) continue;
            let left = i + 1, right = n - 1;
            while (left < right) {
                const sum = nums[i] + nums[left] + nums[right];
                if (sum === 0) {
                    res.push([nums[i], nums[left], nums[right]]);
                    while (left < right && nums[left] === nums[left + 1]) left++;
                    while (left < right && nums[right] === nums[right - 1]) right--;
                    left++; right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n²) — 排序 O(n log n) + 双层扫 O(n²).
- **Space**: O(1) 额外 (原地排序, 结果不算).

## 相关题目

- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — 2 数版, 无序 → hash
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针母题
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 滑窗
- [0027. Remove Element](../0027-remove-element/README.md) — 同向双指针
- [0454. 4Sum II](../../03-hash-table/0454-4sum-ii/README.md) — 4 数组各选一, 分组哈希
- 0016\. 3Sum Closest (待补) — 找最接近 target 的三数之和
- 0018\. 4Sum (待补) — 4 数版, 双层 for + 双指针 + 4 层去重
- 0167\. Two Sum II - Input Array Is Sorted (待补) — 有序 2Sum, 对撞双指针
- 0259\. 3Sum Smaller (待补) — 三数之和 < target 的**计数**
- 0011\. Container With Most Water (待补) — 对撞双指针 + 贪心

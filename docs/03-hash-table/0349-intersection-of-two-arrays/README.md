# 0349. Intersection of Two Arrays / 两个数组的交集

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Set, Array · 哈希集合, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/intersection-of-two-arrays/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Unique intersection of two arrays** → dump `nums1` into hash set; scan `nums2`, matches go into a **result set** (dedupe); flatten to vector.
>
> **中文**: **两数组交集去重** → nums1 入哈希 set, 扫 nums2 命中的塞**结果 set** (天然去重), 转 vector.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给两数组, 返回**交集** (每个元素**只出现一次**), 顺序任意.

**中文**: 求交集 (去重).

## Key Insights

1. **🔑 迭代器区间构造 set / Range-based set construction**

    ```cpp
    unordered_set<int> set1(nums1.begin(), nums1.end());
    ```

    这一行做了两件事: **构造 set** + **天然去重**. 比手写 `for + insert` 快也短.

    > **STL 容器几乎都支持迭代器区间构造** (vector, set, map, deque, ...). 记住这套 API, 少写循环.

2. **🔑 用 set 存结果 = 一步去重 / Result set = one-line dedupe**

    为啥不用 vector 存结果? 因为 **nums2 里可能有重复元素**:

    ```
    nums1 = [1, 2, 3], nums2 = [2, 2, 2]
    若用 vector: [2, 2, 2] ❌ 违反去重
    用 set: {2} ✅
    ```

    > **"结果需要去重" → 结果容器直接选 set**. 而不是最后再手动去重.

3. **🔑 两个 set 分工 / Two sets, two roles**

    - `set1` — 查询表 (nums1 的元素集).
    - `resSet` — 结果表 (**保证输出去重**).

    | 用途 | 需要的操作 | 数据结构 |
    |---|---|---|
    | 判"存在" | `count` / `find` — O(1) | hash set |
    | 收集"结果" | `insert` (含去重) — O(1) | hash set |

    > **"用 hash set 干两件事"** 是这题的核心武器. 不要一个 set 兼任, 语义分开更清晰.

4. **🔑 复杂度: O(n + m) 平均 / Complexity: O(n + m) average**

    - `set1` 构造: O(n).
    - 扫 nums2: O(m).
    - 每次 count / insert: O(1) 平均.
    - **总时间**: O(n + m). 空间: O(n) (set1) + O(min(n, m)) (resSet).

    > 若数据范围**小** (本题 constraints [0, 1000]), 用 `bool[1001]` 更快 (无 hash 开销). 但 hash 版**通用性更好**.

5. **🔑 备选思路 / Alternatives**

    | 方案 | Time | Space | 适用 |
    |---|---|---|---|
    | **hash set** (Yang) | O(n + m) | O(n) | 通用首选 |
    | **排序 + 对撞双指针** | O((n+m) log (n+m)) | O(1) 额外 | 已排序时 |
    | `std::set_intersection` (STL) | 排序后 O(n + m) | O(1) 额外 | 已排序 + 会用 STL |
    | `bool[range]` 计数 | O(n + m + range) | O(range) | 值域小 |

    > 面试**主动列 tradeoff** 加分. Hash 是通用答, 面试官问"若排好序?" → 换双指针.

6. **跟 0350 的区别 / Difference from 0350**

    - **0349**: 交集**去重** (本题).
    - **0350**: 交集**保留最小重复次数** (Multiset 交, 类似 [1002](../1002-find-common-characters/README.md)).

    > 一字之差, 数据结构从 set 变成 Counter. 面试问一定要问"要不要保留重复".

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
            unordered_set<int> set1(nums1.begin(), nums1.end());     // 迭代器区间构造 + 去重
            unordered_set<int> resSet;
            for (int x : nums2) {
                if (set1.count(x)) resSet.insert(x);                 // 命中就塞 (自动去重)
            }
            return vector<int>(resSet.begin(), resSet.end());        // set → vector 也是区间构造
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def intersection(self, nums1: list[int], nums2: list[int]) -> list[int]:
            # set(iterable) 一步去重. set & set 是内建的"求交集" 运算符 — 语义清晰,
            # 内部就是遍历较小的 set + 查另一个, 跟 C++ 手写等价
            # list(set) 把 set 转 list, 顺序无所谓 (题目允许)
            return list(set(nums1) & set(nums2))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums1
     * @param {number[]} nums2
     * @return {number[]}
     */
    var intersection = function(nums1, nums2) {
        // new Set(iter) — 一步去重. 跟 C++ / Python 同源
        const set1 = new Set(nums1);
        const resSet = new Set();
        for (const x of nums2) {
            if (set1.has(x)) resSet.add(x);
        }
        // [...set] spread 展开 Set 成数组; 等价 Array.from(resSet)
        return [...resSet];
    };
    ```

## Complexity

- **Time**: O(n + m) 平均.
- **Space**: O(n + min(n, m)) — 两个 set.

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组基础
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 哈希分桶
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set + "只从头扩"
- [1002. Find Common Characters](../1002-find-common-characters/README.md) — 多集交集 (Counter min)
- 0350\. Intersection of Two Arrays II (待补) — **保留重复次数**, 用 Counter 而非 set
- 1002\. Find Common Characters (已存) — 多词字符交集
- 2215\. Find the Difference of Two Arrays (待补) — 交集 + 差集
- [0202. Happy Number](../0202-happy-number/README.md) — hash set / Floyd 判数字循环

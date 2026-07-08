# 0001. Two Sum / 两数之和

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Array, Hash Table · 数组, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/two-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find two indices summing to target** → **one-pass hash map** `value → index`; for each `x`, check if `target - x` was seen — O(1) lookup. Check-then-insert order guarantees no self-reuse.
>
> **中文**: **找两数下标 (和 = target)** → **一遍扫哈希表** `value → index`; 对当前 x 查 `target - x` 有没有. **先查后插** 保证不重用自己.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 整数数组 + target. 返回**两数下标**使和 = target. 题目保证**恰好一对解**, 同一元素不能用两次.

**中文**: 两数之和 = target 的下标.

## Key Insights

1. **🔑 O(n²) → O(n): 用空间换时间 / Trade space for time**

    朴素暴力: 两层 for 试所有对, O(n²). **哈希表**把"查过去有没有见过某个数" 降到 O(1):

    ```
    for i, x in enumerate(nums):
        c = target - x                    # 需要的搭档
        if c in seen: return [seen[c], i] # 命中
        seen[x] = i                       # 记账
    ```

    > **"两数关系" 类题的通用武器**: 固定当前一个, 查另一个 = "compensating value" 模式. 3Sum / 4Sum 都是它的推广.

2. **🔑 关键不变量: seen 只存"之前的下标" / Invariant: seen holds strictly-earlier indices**

    每次循环开始时, `seen` 里的每个 key 对应下标 `< i`. 所以:

    - 命中 `c` in seen → **一定是前面的数**, 不会撞自己 (`[0,0]` 这种非法结果不可能出现).
    - `nums = [3, 3], target = 6`: i=0 时 seen 空, 存 `3→0`; i=1 时命中 `3→0`, 返 `[0, 1]` ✅.

    > **"先查后插" 顺序不能反**. 反过来就成 `[0, 0]` 撞车.

3. **🔑 为啥用 map (value → index) 不是 set / Why map (value → index), not just set**

    因为题目要**返回下标** 不是值. 若只用 `unordered_set<int>` 判"有没有搭档", 找到后还得再扫一遍数组找它的下标 → 冗余. **map 直接存下标, 一步到位**.

    > 存储什么 = 未来查询需要什么. 想清楚这一步再选数据结构.

4. **🔑 `unordered_map` 不是 `map` / Use `unordered_map`, not `map`**

    | | `map` (红黑树) | `unordered_map` (hash) |
    |---|---|---|
    | 查/插 | O(log n) | **O(1) 平均** |
    | 有序性 | 有 | 无 |
    | 这题需要 | ❌ | ✅ |

    > **本题不需要顺序** → 直接 hash map. 用 `map` 会拿到 O(n log n), 慢一阶.

5. **🔑 图解 / Trace**

    `nums = [2, 7, 11, 15], target = 9`:

    ```mermaid
    graph LR
        A["i=0, x=2<br/>seen={}<br/>miss → seen={2:0}"] --> B["i=1, x=7<br/>c=2 ✓ seen<br/>return [1, 0]"]
        style B fill:#c8e6c9
    ```

    第二步命中, 一遍搞定.

6. **🔑 变种预告 / Variant preview**

    - 若数组**已排序**: 用**对撞双指针**, O(1) 空间. → 0167 Two Sum II.
    - **数据结构进阶** 到 BST: → 0653.
    - **返回所有对 / 3 个数**: → 0015 3Sum (排序 + 对撞 + 去重).
    - **要求原地 O(1) 空间 + 不排序**: 不可能, 排除.

7. **复杂度 O(n) 时间, O(n) 空间 / Linear time, linear space**

    - Time: 一遍扫, hash O(1) 摊销.
    - Space: 最坏 map 存 n - 1 个元素.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> twoSum(vector<int>& nums, int target) {
            unordered_map<int, int> um;                             // value → index
            for (int i = 0; i < (int)nums.size(); i++) {
                if (um.find(target - nums[i]) != um.end()) {
                    return {i, um[target - nums[i]]};               // 命中: 返 (当前, 之前)
                }
                um[nums[i]] = i;                                    // 记账
            }
            return {};
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def twoSum(self, nums: list[int], target: int) -> list[int]:
            # dict = hash map. enumerate 一次拿 index+value, 比 for i in range(len(nums)) 干净
            seen: dict[int, int] = {}
            for i, x in enumerate(nums):
                # 海象运算符 := (Python 3.8+): 赋值 + 表达式一步. 相当于 C++ if (auto c = ...; ...)
                # 语义: 算 c 后立刻检查 c in seen
                if (c := target - x) in seen:
                    return [seen[c], i]
                seen[x] = i
            return []
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number[]}
     */
    var twoSum = function(nums, target) {
        // Map 而不是 Object — Map 保留数字 key 类型 (Object 会把 key 转 string), 也更接近 C++ unordered_map
        const seen = new Map();
        for (let i = 0; i < nums.length; i++) {
            const c = target - nums[i];
            // .has() O(1) 平均查, 跟 C++ .find() != .end() 等价
            if (seen.has(c)) return [seen.get(c), i];
            seen.set(nums[i], i);
        }
        return [];
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫, hash 操作 O(1) 平均.
- **Space**: O(n) — 最坏 map 装 n - 1 个元素.

## 易错点

- **先查后插的顺序**: `if (查到) return; 记账;` — 反了会用同一元素两次 (`nums=[3,3], target=6` 变 `[0,0]`).
- **用 `unordered_map` 不是 `map`**: C++ 里 `map` 是 O(log n), 拖慢一阶.

## 相关题目

- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 哈希 / 计数数组基础
- [0349. Intersection of Two Arrays](../../03-hash-table/0349-intersection-of-two-arrays/README.md) — hash set 交集
- [0128. Longest Consecutive Sequence](../../03-hash-table/0128-longest-consecutive-sequence/README.md) — hash set + "只从头扩"
- [0202. Happy Number](../../03-hash-table/0202-happy-number/README.md) — hash set 判循环
- 0167\. Two Sum II - Input Array Is Sorted (待补) — 对撞双指针, O(1) 空间
- 0653\. Two Sum IV - Input is a BST (待补) — BST + 中序 or hash
- 0015\. 3Sum (待补) — 排序 + 对撞双指针 + 去重
- 0018\. 4Sum (待补) — 二层枚举 + 3Sum
- 0454\. 4Sum II (待补) — 双 hash 分组, 拆解 4 数为 (2 + 2)
- 0170\. Two Sum III - Data structure design (待补) — 设计题

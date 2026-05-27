# 0496. Next Greater Element I / 下一个更大元素 I

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Monotonic Stack, Hash Table · 单调栈, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/next-greater-element-i/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个**不重复** 数组 `nums1` (子集) 和 `nums2`. 对 `nums1[i]`, 在 `nums2` 里找到它**对应位置**, 返回 `nums2` 中它**之后第一个比它大** 的元素. 没有则返 `-1`.

**中文**: `nums1` 是 `nums2` 的子集 (元素都不重复). 对 `nums1` 每个元素, 找它在 `nums2` 中的下一个更大元素, 没有则 -1.

## Key Insights

1. **🔑 跟 [0739 每日温度](../0739-daily-temperatures/README.md) 同模板, 单调栈最基础 / Same Next-Greater-Element template**

    跟 0739 完全同结构, 但两个差别:

    | | [0739](../0739-daily-temperatures/README.md) | **0496 (本题)** |
    |---|---|---|
    | 栈存什么 | **索引** (要算距离 `i - stk.top()`) | **值** (要存进 map) |
    | 输出什么 | 距离数组 | "值 → 下一个更大值" map, 再按 nums1 查 |
    | 输入 | 单数组 | 两数组 (nums1 ⊆ nums2) |

    > **栈存索引 vs 存值** 是单调栈题型的两大变种, 看输出需求决定. 0739 要距离选索引, 0496 要值映射选存值.

2. **状态: `m[x] = nums2 中 x 之后第一个更大元素` / Map "value → next greater"**

    对 nums2 跑一遍单调栈预处理 — 每个元素被弹出时, 它"找到了下一个更大", 记入 map.

    > 这是"**预处理大集合, 按需查小集合**" 的经典套路. nums1 只是查询接口.

3. **🔑 弹栈即"找到下一个更大" / Pop = found next greater**

    维护单调递减栈. 来 `n`:

    - 栈顶 `< n` → 栈顶的"下一个更大" 就是 `n` → 记 `m[stk.top()] = n`, 弹出.
    - 直到栈空或栈顶 ≥ n, 把 `n` 压栈.

    扫完 nums2, 栈里剩的元素都"没有下一个更大" → 它们**不会出现在 map 里**, 查时返 -1.

4. **`unordered_map` 查 -1 兜底 / Missing → -1**

    Yang 用 `m.find(n)` + 三元判 end:

    ```cpp
    auto it = m.find(n);
    res.push_back(it != m.end() ? it->second : -1);
    ```

    等价写法 (略短, 但慢一些 — 两次查):

    ```cpp
    res.push_back(m.count(n) ? m[n] : -1);
    ```

    Python 用 `m.get(n, -1)` 一行更干净, JS 用 `m.get(n) ?? -1`. C++17 起也有 `m.contains(n)` 但仍需第二次访问取值.

5. **复杂度 O(n + m) / Complexity**

    - 预处理 nums2: O(n2), 每元素入栈出栈各一次.
    - 查询 nums1: O(n1), 每元素 O(1) 哈希查.
    - 总 O(n1 + n2). 比"对每个 nums1 元素都扫 nums2" 的 O(n1 × n2) 暴力快得多.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
            unordered_map<int, int> m;
            stack<int> stk;                                        // 栈存值
            // 对 nums2 跑单调栈, 弹栈时记 map
            for (int n : nums2) {
                while (!stk.empty() && n > stk.top()) {
                    m[stk.top()] = n;
                    stk.pop();
                }
                stk.push(n);
            }
            // nums1 查 map, 缺则 -1
            vector<int> res;
            res.reserve(nums1.size());
            for (int n : nums1) {
                auto it = m.find(n);
                res.push_back(it != m.end() ? it->second : -1);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def nextGreaterElement(self, nums1: list[int], nums2: list[int]) -> list[int]:
            # m[x] = nums2 中 x 后第一个更大元素
            m, stk = {}, []
            for n in nums2:
                while stk and n > stk[-1]:
                    m[stk.pop()] = n                               # 弹栈时确定答案
                stk.append(n)
            # dict.get(key, default) 一行查 + 缺省 -1, 比 C++ if-find 干净
            return [m.get(n, -1) for n in nums1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums1
     * @param {number[]} nums2
     * @return {number[]}
     */
    var nextGreaterElement = function(nums1, nums2) {
        const m = new Map();
        const stk = [];
        for (const n of nums2) {
            while (stk.length && n > stk[stk.length - 1]) {
                m.set(stk.pop(), n);
            }
            stk.push(n);
        }
        // ?? 是 nullish coalescing: undefined/null 时取右边
        return nums1.map(n => m.get(n) ?? -1);
    };
    ```

## Complexity

- **Time**: O(n1 + n2).
- **Space**: O(n2) — map + stack.

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 同模板, 栈存索引求距离
- [0503. Next Greater Element II](../0503-next-greater-element-ii/README.md) — 循环数组版, "走两圈" 处理环
- 0556\. Next Greater Element III (待补) — 数字字典序下一个更大, 不是数组
- [0901. Online Stock Span](../0901-online-stock-span/README.md) — 流式版单调栈, 栈存 (price, span) 块
- [0316. Remove Duplicate Letters](../0316-remove-duplicate-letters/README.md) — 单调栈 + 贪心字典序
- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 单调栈经典进阶

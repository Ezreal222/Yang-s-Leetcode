# 0503. Next Greater Element II / 下一个更大元素 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Array · 单调栈, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/next-greater-element-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给**循环数组** `nums` (末尾接回开头). 对每个位置, 找下一个更大元素 (按循环顺序). 没有则 `-1`.

**中文**: 循环数组 (末尾接首位), 求每个位置的下一个更大元素, 没有返 -1.

## Key Insights

1. **🔑 循环数组处理: "走两圈" 技巧 / Circular array via 2n iteration**

    跟 [0496](../0496-next-greater-element-i/README.md) / [0739](../0739-daily-temperatures/README.md) 的"单向数组" 不同, 这题是环 — 一个位置的"下一个更大" 可能**绕回** 在它前面.

    **关键技巧**: 循环 `i` 从 0 走到 **2n-1**, 用 `nums[i % n]` 访问. 等价于把数组**概念上复制一份接在后面** — 走完两圈, 每个位置都"看到" 整个环.

    > **`for i in 0..2n, use arr[i % n]`** 是循环数组的标准套路. 同思想还可用 `arr + arr` 实际拼接, 但 `% n` 更省内存.

2. **🔑 只在第一遍 push, 第二遍只 pop / Push only in first pass**

    Yang 的关键观察:

    ```cpp
    if (i < n) stk.push(i);     // 只第一遍 push
    ```

    第一遍把所有原始索引 `0..n-1` 入栈一次. 第二遍 (`i ∈ [n, 2n-1]`) 只**消费**这些尚未找到答案的索引 (那些"次大要绕回去找" 的). 不 push 防重复入栈.

    > **第二遍是"补刀"**, 让没找到答案的索引看到环对岸的更大元素. 它们答案确认完成自然弹出, 漏的留在栈里 → res 保持初值 -1.

3. **状态: 栈存索引 (跟 [0739](../0739-daily-temperatures/README.md) 同款) / Stack holds indices**

    要往 `res[idx]` 赋值, 必须知道哪个位置在等答案 — 存索引而非值. 跟 [0496](../0496-next-greater-element-i/README.md) 的"栈存值 + map" 形成对比.

4. **`res` 默认 `-1` 兜底 / Default -1 for unresolved**

    `vector<int> res(n, -1)` 初始全 -1. 找到答案的位置覆盖, 没找到的保留 -1 — 对应"环里都没有更大".

5. **复杂度仍是 O(n) / Still O(n)**

    每个原始索引最多入栈一次, 出栈一次. 2n 循环里, push 只发生 n 次, pop 最多 n 次. **总 O(n)** — 循环数组的常数翻倍但量级不变.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> nextGreaterElements(vector<int>& nums) {
            int n = nums.size();
            vector<int> res(n, -1);                                // 默认 -1
            stack<int> stk;
            for (int i = 0; i < 2 * n; i++) {                      // 走两圈
                int cur = nums[i % n];                             // 循环访问
                while (!stk.empty() && cur > nums[stk.top()]) {
                    res[stk.top()] = cur;
                    stk.pop();
                }
                if (i < n) stk.push(i);                            // ⚠ 只第一遍 push
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def nextGreaterElements(self, nums: list[int]) -> list[int]:
            n = len(nums)
            res = [-1] * n
            stk = []
            for i in range(2 * n):                                 # 走两圈
                cur = nums[i % n]
                while stk and cur > nums[stk[-1]]:
                    res[stk.pop()] = cur
                if i < n:                                          # 只第一遍 push
                    stk.append(i)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number[]}
     */
    var nextGreaterElements = function(nums) {
        const n = nums.length;
        const res = new Array(n).fill(-1);
        const stk = [];
        for (let i = 0; i < 2 * n; i++) {
            const cur = nums[i % n];
            while (stk.length && cur > nums[stk[stk.length - 1]]) {
                res[stk.pop()] = cur;
            }
            if (i < n) stk.push(i);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 2n 循环, 每索引入栈出栈各一次.
- **Space**: O(n) — 栈最坏存全部.

## 相关题目

- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — 单向数组 + 子集查询
- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 单向数组 + 距离
- 0556\. Next Greater Element III (待补) — 数字字典序下一个更大
- 0901\. Online Stock Span (待补) — 流式数据 + 单调栈
- 0918\. Maximum Sum Circular Subarray (待补) — 循环数组 + Kadane, 同款"走两圈/拆环为链" 思路
- 0084\. Largest Rectangle in Histogram (待补) — 单调栈经典进阶

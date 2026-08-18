# 0053. Maximum Subarray / 最大子数组和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, DP, Divide and Conquer · 贪心, 动态规划, 分治
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-subarray/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given an integer array `nums`, find the contiguous subarray (containing at least one number) with the largest sum, and return that sum.

**中文**: 给一个整数数组 `nums`, 找出连续子数组 (至少含一个数) 中和最大的, 返回该和.

## 思路

### Core idea

**贪心 (Kadane's) 一遍扫**:

- 维护 `sum` = 当前以 nums[i] 结尾的候选最大子数组和.
- 每步 `sum += nums[i]`, 然后 `res = max(res, sum)`.
- 若 `sum < 0`, 重置 `sum = 0` — 任何后续从这里开始的子数组都会被这个负 prefix 拖累, 不如丢掉重起.

### Key Insights

1. **`sum < 0 重置` 的贪心正确性 / Why dropping a negative prefix is optimal**

    假设当前 prefix sum 是负数. 接下来无论遇到什么序列, "保留这个负 prefix" 一定不如"从下一个元素重新开始" — 因为加一个负数永远是亏的.

    形式化: 若 `prefix_sum < 0`, 那么 `prefix_sum + suffix < suffix`, 所以丢 prefix 严格更优. 这是局部最优可推全局最优的典型贪心.

2. **顺序: `+=` → `max` → `if reset` / Order is load-bearing**

    必须先累加, **然后** 更新 res, **最后** 才判 reset. 顺序反了会漏 single-element 最大值. 例: `[-1]` — 加完后 sum = -1, max(res, -1) = -1 (正确), 然后才 reset. 把 reset 提前会让 res 永远是 INT_MIN.

3. **`res = INT_MIN` 不是 0 / Initial value matters for all-negative input**

    全负数组 (如 `[-3, -1, -2]`), 答案应该是 -1 (最大的单元素), 不是 0 (空子数组不算). 初始 INT_MIN 保证全负 case 也能取到正确最大值.

    > Python 用 `float('-inf')`, JS 用 `-Infinity`.

4. **跟 DP 版 (Kadane's 标准式) 的关系 / DP equivalent**

    ```cpp
    int dp = nums[0], res = nums[0];
    for (int i = 1; i < n; i++) {
        dp = max(nums[i], dp + nums[i]);    // 要么续上之前, 要么从我开始
        res = max(res, dp);
    }
    ```
    `dp[i]` = "以 i 结尾的最大子数组和" — 跟贪心版的 `sum` 等价. 贪心版的"sum < 0 重置" 就是 DP 转移里"要么续要么重起" 的另一种表述.

5. **分治版 O(n log n) / Divide-and-conquer alternative**

    递归求 (左半最大 / 右半最大 / 跨中线最大). 经典面试题但贪心更优. 知道分治存在即可, 不必背代码.

6. **不要返回子数组本身, 只要 sum**

    题目只要 sum. 如果题目变体要返回起止 index, 在 `sum < 0 重置时` 记 start = i + 1, 每次更新 res 时记 end. 多两个变量.

### 一句话总结

**贪心: `sum += nums[i]` → `res = max(res, sum)` → `sum < 0 重置`. 三步固定顺序, INT_MIN 处理全负 case.**

### 图解

`nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`:

```
i:   0   1   2   3   4   5   6   7   8
num: -2  1   -3  4   -1  2   1   -5  4
sum: -2  1   -2  4   3   5   6   1   5      (sum<0 重置点: i=0 后, i=2 后)
res: -2  1   1   4   4   5   6   6   6      → 6 ([4,-1,2,1])
```

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to find the **contiguous subarray** with the **largest sum** in `nums`. Let me confirm a few things:"

- "**Contiguous**, right? Not subsequence." *(yes.)*
- "Must the subarray be **non-empty**? So if all numbers are negative, I return the least-negative single element, not zero?" *(yes — non-empty.)*
- "Any constraint on `n`? Just to pick the right complexity." *(usually up to 10⁵ — O(n) needed.)*
- "**Value range** — could sums overflow?" *(usually 32-bit int fits, but for very long arrays I'd use `long long`.)*

### 2. Brainstorm approaches (1 min)

> "Three approaches worth mentioning, from slowest to fastest.
>
> **Approach 1 — brute force**: try every `(i, j)` pair, sum the subarray. O(n³) with naive sum, O(n²) with a running sum. Way too slow for 10⁵.
>
> **Approach 2 — divide and conquer**: recurse on left half, right half, and 'crossing the middle' — take the max. O(n log n). Classic textbook, but there's better.
>
> **Approach 3 — greedy / Kadane's algorithm**: single pass, O(n). At each index, either **extend** the current subarray or **restart** from here. If the running sum ever goes negative, throw it away — carrying a negative prefix into the future always hurts.
>
> Approach 3 is O(n), O(1) space. That's my pick."

### 3. Sketch the algorithm before coding (1 min)

> "The greedy state is a single running variable `sum`. On each element:
>
> 1. **Add** it to `sum`.
> 2. **Update** the best answer: `res = max(res, sum)`.
> 3. **Reset** `sum = 0` if it went negative.
>
> Two design choices worth flagging out loud:
>
> - **The three-step order is load-bearing**: `+= → max → if reset`. If I reset *before* updating `res`, I miss the case where the current element itself is the peak — think `[-1]`, the answer should be `-1`, not some sentinel.
> - **Initial `res = INT_MIN`, not 0.** Because 'non-empty' means all-negative arrays must return their max element, not 0."

### 4. Code while narrating (2 min)

```cpp
int maxSubArray(vector<int>& nums) {
    int sum = 0, res = INT_MIN;
    for (int x : nums) {
        sum += x;                  // 1. extend
        res = max(res, sum);       // 2. record best
        if (sum < 0) sum = 0;      // 3. reset if negative
    }
    return res;
}
```

> "Seven lines. The whole insight lives in step 3 — the greedy 'drop a negative prefix' choice."

### 5. Prove the greedy step (30s)

> "One thing worth saying out loud, since greedy always needs justification: **why is resetting when `sum < 0` correct?**
>
> If my prefix sum is negative, then for *any* future suffix `S`, `prefix + S < S`. So starting fresh from the next element is strictly better than carrying the negative prefix forward. That's the local-optimal-implies-global-optimal argument for Kadane's."

### 6. Trace an example (1 min)

> "Let me trace `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`:
>
> | i | nums[i] | sum after `+=` | res | reset? |
> |---|---|---|---|---|
> | 0 | -2 | -2 | **-2** | yes → 0 |
> | 1 | 1 | 1 | 1 | no |
> | 2 | -3 | -2 | 1 | yes → 0 |
> | 3 | 4 | 4 | 4 | no |
> | 4 | -1 | 3 | 4 | no |
> | 5 | 2 | 5 | 5 | no |
> | 6 | 1 | 6 | **6** | no |
> | 7 | -5 | 1 | 6 | no |
> | 8 | 4 | 5 | 6 | no |
>
> Answer: **6**, from the subarray `[4, -1, 2, 1]`. The reset points (after i=0 and i=2) each represent 'that prefix was hurting; drop it.'"

### 7. Complexity + follow-ups (1 min)

> "**Time O(n)**, **space O(1)** — one pass, three scalar variables.
>
> A few natural follow-ups:
>
> 1. **DP framing**: this is equivalent to Kadane's DP where `dp[i] = max(nums[i], dp[i-1] + nums[i])` — 'extend or restart.' The greedy `sum < 0 reset` is that same choice, expressed in-place.
>
> 2. **Return the subarray indices** (not just the sum): add two variables `start, end`. Set `start = i + 1` at each reset, and update `end = i` whenever `res` improves.
>
> 3. **Circular version** (0918): rotate through the end. Trick — either the answer is a normal max subarray, *or* it wraps around, which is `total_sum − min_subarray`. Take max of the two, with an edge case when all values are negative.
>
> 4. **Product version** (0152): can't use Kadane directly because a big negative × a big negative = big positive. Track both current max **and** current min at each step.
>
> Any follow-ups you'd like me to code?"

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxSubArray(vector<int>& nums) {
            int sum = 0, res = INT_MIN;
            for (int i = 0; i < (int)nums.size(); i++) {
                sum += nums[i];                            // 1. 先累加
                res = max(res, sum);                       // 2. 再更新答案
                if (sum < 0) sum = 0;                      // 3. 最后重置 (顺序必须固定)
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxSubArray(self, nums: list[int]) -> int:
            cur = 0
            res = float('-inf')                            # -∞, 等价 C++ INT_MIN
            for n in nums:
                cur += n
                res = max(res, cur)
                if cur < 0:
                    cur = 0
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var maxSubArray = function(nums) {
        let cur = 0, res = -Infinity;                      // 等价 INT_MIN
        for (const n of nums) {
            cur += n;
            res = Math.max(res, cur);
            if (cur < 0) cur = 0;
        }
        return res;
    };
    ```

### 替代: DP 版 (Kadane's 标准式)

```cpp
int maxSubArray(vector<int>& nums) {
    int dp = nums[0], res = nums[0];
    for (int i = 1; i < (int)nums.size(); i++) {
        dp = max(nums[i], dp + nums[i]);                   // 要么从 nums[i] 重起, 要么续上
        res = max(res, dp);
    }
    return res;
}
```
跟贪心等价, 状态意义更直白 ("以 i 结尾的最大子数组和"). 二选一.

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 易错点

- **`res = INT_MIN` 不是 0**: 全负数组 (如 `[-3,-2,-1]`) 答案是 -1, 不是 0. 子数组必须非空.
- **顺序固定: `+=` → `max` → `if reset`**: 顺序反了 (例如先 reset 后 max) 会漏 single-element 最大值. 三步是这题的"小心思", 易顺手写错.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门
- [0376. Wiggle Subsequence](../0376-wiggle-subsequence/README.md) — 同款一遍扫贪心
- [0152. Maximum Product Subarray](../../10-dp/0152-maximum-product-subarray/README.md) — 乘积版, 维护 max/min 两个值 (在 §10)
- 0918\. Maximum Sum Circular Subarray (待补) — 环形版, 拆成"普通最大" + "总和 - 最小"
- 0978\. Longest Turbulent Subarray (待补) — 同款一遍扫, 状态是"上升/下降" 翻转
- 1567\. Maximum Length of Subarray With Positive Product (待补) — 一遍扫维护正负长度

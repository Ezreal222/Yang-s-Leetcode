# 0300. Longest Increasing Subsequence / 最长递增子序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Binary Search, Patience Sorting · 动态规划, 二分, 耐心排序
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-increasing-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: 给整数数组 `nums`, 返回**最长严格递增子序列** 的长度.

**中文**: 给 `nums`, 求最长**严格递增子序列** 的长度. 子序列不要求连续.

## Key Insights

1. **🔑 状态定义: `dp[i] = 以 nums[i] 结尾的最长 LIS` (强制选 i) / dp[i] = LIS ending at i**

    跟 [0198 House Robber](../0198-house-robber/README.md) 的"到 i 的最优" 不同, LIS 必须**强制以 i 结尾** — 否则没法判定"下一个元素能不能接上 i".

    > **"以 i 结尾" 状态** 是序列 DP 的核心模式. 凡是问"最长子序列 / 子数组" 一类, 多半要这么定. 跟"到 i 的最优" (不强制选 i) 的区别要分清.

2. **答案不是 `dp[n-1]`, 而是 `max(dp[*])` / Answer scans all dp values**

    因为 `dp[i]` 强制以 i 结尾, 最长 LIS 可能在任何位置结束. 需要扫一遍取最大. 这是"以 i 结尾" 状态的代价.

3. **v1 转移: `dp[i] = max(dp[j] + 1)` 对所有满足 `nums[j] < nums[i]` 的 j < i / O(n²) transition**

    用[最后一步思维](../topic-dp-thinking-process.md): 以 i 结尾的 LIS, **倒数第二个元素** 在哪? 枚举所有可能的 `j < i` 且 `nums[j] < nums[i]`, 接到 i 的 LIS 长度就是 `dp[j] + 1`. 取最大.

    初始: `dp[i] = 1` (至少自己单独一个就是长度 1 的 LIS).

4. **🔑 v2 O(n log n) 神操作: tails 数组 + lower_bound / Patience sorting**

    维护一个 `tails` 数组, **`tails[k] = 所有长度为 k+1 的递增子序列中, 末位最小的那个值`**. 关键性质:

    - **tails 始终严格递增** (induction proof)
    - **`tails.size()` 就是当前 LIS 长度**, 但 **tails 本身不是 LIS** (只是各长度的最优末位)

    扫每个 num:

    - `lower_bound(tails, num)` 找第一个 ≥ num 的位置 `it`.
    - 若 `it == end`: num 比所有现有末位都大 → 可以接到最长 IS 后 → `push_back(num)`, LIS 长度 +1.
    - 否则: `*it = num` (用 num 替换那个位置, 让对应长度的"末位" 变小, 给后续留更多空间).

    > **"替换" 那一步是反直觉的核心**. 替换不影响当前 LIS 长度, 但让未来更容易接更长的 IS — 因为末位变小了. 这是 patience sorting 的精髓.

5. **`lower_bound` (严格递增) vs `upper_bound` (非递减) / Lower vs upper**

    本题要求**严格递增**, 所以用 `lower_bound` (找第一个 ≥ num, 把等于的也替换掉, 不允许重复).

    若题目改成"最长非递减子序列", 改用 `upper_bound` (找第一个 > num, 等于的保留, 允许追加).

    > 一字之差, 严格/非严格切换. 记住口诀: **"严格递增 → lower_bound, 非递减 → upper_bound"**.

6. **⚠ tails 不是真实 LIS / Tails ≠ actual subsequence**

    打印 tails 出来不一定是合法子序列 — 它只是"各长度的最优末位 snapshot". 例: `nums = [10, 9, 2, 5, 3, 7]` 最终 `tails = [2, 3, 7]`, 长度 3 对; 但 `2, 3, 7` 在 nums 里的实际顺序是 `2, 5, 3, 7` (或 `2, 3, 7`, 这次刚好对). 真要重构 LIS, 得额外维护 parent 指针 + 每个元素被加入 tails 时的索引.

## Solution

=== "C++"
    === "v1: O(n²) DP (易懂)"
        ```cpp
        class Solution {
        public:
            int lengthOfLIS(vector<int>& nums) {
                int n = nums.size();
                vector<int> dp(n, 1);                              // 每个元素自身就是长度 1
                int result = 1;
                for (int i = 1; i < n; i++) {
                    for (int j = 0; j < i; j++) {
                        if (nums[i] > nums[j]) {                   // 严格递增
                            dp[i] = max(dp[i], dp[j] + 1);
                        }
                    }
                    result = max(result, dp[i]);
                }
                return result;
            }
        };
        ```

    === "v2 推荐: O(n log n) patience sorting"
        ```cpp
        class Solution {
        public:
            int lengthOfLIS(vector<int>& nums) {
                vector<int> tails;                                 // tails[k] = 长度 k+1 IS 的最小末位
                for (int num : nums) {
                    auto it = lower_bound(tails.begin(), tails.end(), num);
                    if (it == tails.end()) tails.push_back(num);   // 接到最长后, 长度 +1
                    else                   *it = num;              // 替换: 让对应长度末位变小
                }
                return tails.size();                               // ⚠ tails 不是真实 LIS
            }
        };
        ```

=== "Python"
    ```python
    from bisect import bisect_left

    class Solution:
        def lengthOfLIS(self, nums: list[int]) -> int:
            # v2 O(n log n): tails 数组 + 二分
            # bisect_left = C++ lower_bound, 找第一个 ≥ num 的位置
            tails = []
            for num in nums:
                idx = bisect_left(tails, num)
                if idx == len(tails):
                    tails.append(num)                              # 接到最长后
                else:
                    tails[idx] = num                               # 替换
            return len(tails)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var lengthOfLIS = function(nums) {
        // JS 没有原生 lower_bound, 手写一个
        const lowerBound = (arr, target) => {
            let lo = 0, hi = arr.length;
            while (lo < hi) {
                const mid = (lo + hi) >> 1;
                if (arr[mid] < target) lo = mid + 1;
                else                   hi = mid;
            }
            return lo;
        };

        const tails = [];
        for (const num of nums) {
            const idx = lowerBound(tails, num);
            if (idx === tails.length) tails.push(num);             // 接到最长后
            else                       tails[idx] = num;           // 替换
        }
        return tails.length;
    };
    ```

## Complexity

- **Time**: O(n²) (v1) / O(n log n) (v2).
- **Space**: O(n).

## Interview Walkthrough

7-step speak-out-loud script for the LIS interview.

### 1. Clarify

> "Just to confirm: strictly increasing, right? So `[1, 3, 3, 5]` — the LIS is 3 (`1, 3, 5`), not 4. And it's a subsequence, so elements don't have to be contiguous — I can skip. Any constraints on n? If it's up to 10⁴ or 10⁵, I want to aim for O(n log n)."

Nail down: **strict vs non-strict** (changes lower_bound / upper_bound), **subsequence vs subarray** (subsequence = skip allowed), **return length or the actual sequence** (they usually want length).

### 2. Brainstorm

> "Two obvious approaches. **Brute force** — try every subsequence, check increasing, O(2ⁿ). Dies at n = 20. **DP** — for each i, `dp[i]` = longest LIS ending at i. Transition: look at every j < i, if `nums[j] < nums[i]`, `dp[i] = max(dp[i], dp[j] + 1)`. That's O(n²). Good enough for n = 2500.
>
> "There's also a clever **O(n log n)** approach using a `tails` array + binary search — patience sorting. Want me to code the DP first for clarity, then optimize?"

**Signal you know both.** Interviewer picks. If they say "just do the fast one", go straight to v2.

### 3. Sketch

For v1 DP:

> "`dp[i]` = LIS length ending at i, forced to include i. Init all to 1 (self). For each i from 1 to n-1, scan j from 0 to i-1: if `nums[j] < nums[i]`, `dp[i] = max(dp[i], dp[j] + 1)`. Answer is `max(dp)`, NOT `dp[n-1]` — because LIS can end anywhere."

For v2 patience:

> "I keep a `tails` array where `tails[k]` = smallest possible tail of any LIS of length `k+1`. Key invariants: **tails is strictly increasing**, and **`tails.size()` equals current LIS length**.
>
> "For each num: binary-search the first `tails[k] ≥ num`. If found: **replace** it with num (shorter tail = more room later). If not found: **append** num (LIS just grew by 1)."

### 4. Code + narrate

Pick v2 unless interviewer asks for DP first:

```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int num : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end()) tails.push_back(num);
        else                   *it = num;
    }
    return tails.size();
}
```

Narrate: **"`lower_bound` for strict increasing — I want to replace equal values, not stack them. If it were non-decreasing, I'd use `upper_bound`."** That one sentence proves you know the edge.

### 5. Trace

`nums = [10, 9, 2, 5, 3, 7, 101, 18]`:

| step | num | tails before | action | tails after |
|---|---|---|---|---|
| 1 | 10 | [] | append | [10] |
| 2 | 9  | [10] | replace idx 0 | [9] |
| 3 | 2  | [9] | replace idx 0 | [2] |
| 4 | 5  | [2] | append | [2, 5] |
| 5 | 3  | [2, 5] | replace idx 1 | [2, 3] |
| 6 | 7  | [2, 3] | append | [2, 3, 7] |
| 7 | 101 | [2, 3, 7] | append | [2, 3, 7, 101] |
| 8 | 18 | [2, 3, 7, 101] | replace idx 3 | [2, 3, 7, 18] |

Answer: 4. **Callout: `[2, 3, 7, 18]` is NOT the actual LIS** — the real one is `[2, 5, 7, 101]` or `[2, 3, 7, 101]`. Tails is only the length invariant.

### 6. Complexity

> "Time is O(n log n) — for each of n elements, one binary search O(log n). Space O(n) for tails. Compared to v1 DP: n² → n log n, which for n = 10⁵ is 10¹⁰ vs 10⁶·⁵ — night and day."

### 7. Follow-ups

- **"Return the actual LIS, not just length"**: augment with parent pointers — each element that goes into tails records its predecessor's index; at the end, walk back from the last-appended element. Answer becomes O(n log n) time, O(n) extra space.
- **"Non-decreasing version"**: switch `lower_bound` to `upper_bound`. One-token change.
- **"Number of LIS"** ([0673](待补)): DP with `(length, count)` per index, aggregate max length + summed count.
- **"2D LIS — Russian Doll Envelopes"** ([0354](待补)): sort by width asc, **tie-break height desc** (to prevent same-width envelopes from stacking), then run LIS on the height array.
- **"Longest chain of pairs / longest bitonic"**: same skeleton, different comparator.

## 相关题目

- 0673\. Number of Longest Increasing Subsequence (待补) — LIS 个数 (DP + 计数)
- 0354\. Russian Doll Envelopes (待补) — 二维 LIS, 先按宽排序再对高跑 LIS
- 0334\. Increasing Triplet Subsequence (待补) — 简化版"长度 ≥ 3 的 LIS 存在?", 一遍扫
- [0674. Longest Continuous Increasing Subsequence](../0674-longest-continuous-increasing-subsequence/README.md) — 连续版, 一遍扫
- 0053\. Maximum Subarray — 子数组连续版 → [§09 0053](../../09-greedy/0053-maximum-subarray/README.md)
- [0718. Maximum Length of Repeated Subarray](../0718-maximum-length-of-repeated-subarray/README.md) — 双序列 DP 入门 (子数组版)
- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — 双序列 DP, LIS 的"两数组" 兄弟 (子序列版)
- [0446. Arithmetic Slices II - Subsequence](../0446-arithmetic-slices-ii-subsequence/README.md) — 序列 DP 进阶: 状态从一维升级到"数组 × 哈希表"
- [§10 DP 思维流程 — "以 i 结尾" 状态模式](../topic-dp-thinking-process.md)

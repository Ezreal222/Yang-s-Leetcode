# 1005. Maximize Sum Of Array After K Negations / K 次取反后最大化的数组和

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Greedy, Sort · 贪心, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/maximize-sum-of-array-after-k-negations/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given integer array `nums` and integer `k`, perform exactly `k` operations, each negates one element (same element can be negated multiple times). Return the maximum possible sum after k operations.

**中文**: 给整数数组 `nums` 和整数 `k`, 必须执行恰好 `k` 次操作, 每次把某个元素取反 (同一元素可以被取反多次). 求 k 次后数组最大和.

## 思路

### Core idea

**两阶段贪心**:

1. **按绝对值降序 sort**: 让绝对值大的元素排前面.
2. **Phase 1**: 从前往后扫, 见负数就翻, `k--`. 优先翻绝对值大的负数 (每翻一个赚 2|x|).
3. **Phase 2**: 扫完后 k 还有剩, 全部耗在**末尾元素** (= 排序后绝对值最小的). 偶数次相当于不翻, 只关心奇偶.

最后 `sum(nums)`.

### Key Insights

1. **按绝对值降序 sort 的贪心收益 / Why |x| desc, not asc**

    翻一个负数 `-x` (x > 0) 收益是 `2x` (从 -x 变成 +x, 总和增加 2x). 所以应该**优先翻绝对值大的负数**.

    用绝对值升序排会先翻小的负数, 浪费宝贵的 k. 倒过来排刚好对.

2. **Phase 2: 剩余 k 全压最小 |x|, 只看奇偶 / Dump remainder on smallest |x| via parity**

    翻完所有负数后, 数组全正 (或全部翻不动了). 剩余 k 次必须落在某个元素上, 翻**最小** |x| 损失最小. 翻偶数次等于不翻, 所以**只需要看 `k % 2`** — 奇数翻一次, 偶数不动.

    > "末尾元素 = 最小 |x|" 因为我们按 |x| 降序 sort 了.

3. **`if (k <= 0) break` 早停 / Exit when k runs out**

    若负数比 k 多, 我们 k 耗光后还要停, 不能继续翻别的负数. 否则会把"赚的负数" 变回负数, 反亏.

4. **跟 [0455 / 0376 / 0053 / 0122 / 0055] 的对照 / Two-phase greedy is a new flavor**

    | 题 | 贪心结构 |
    |---|---|
    | 0455 | 双 sort + 两指针配对 |
    | 0376 | 一遍扫数峰谷 |
    | 0053 | 一遍累加 + reset |
    | 0122 | 一遍收所有正差 |
    | 0055 | 维护可达范围 |
    | **1005 (本题)** | **sort + 阶段化操作 + 余数处理** |

    "排序后分阶段处理" 是贪心题里常见的另一形态. 跟单 pass 不同, 需要先全局排序定优先级.

5. **复杂度 O(n log n) / Sort dominates**

    sort 主导. Phase 1/2 都是 O(n). Sum 也是 O(n). 整体 O(n log n).

### 一句话总结

**按 |x| 降序 sort. 优先翻大负数 (k--). k 还有剩就全压最小 |x|, 只看奇偶. 最后求和.**

### 图解

`nums = [-8, 3, -5, -3, -1, 2]`, `k = 6`. 按 |x| 降序: `[-8, -5, 3, -3, 2, -1]`.

```
i=0 -8 (负) → 8.  k=5  → [8, -5, 3, -3, 2, -1]
i=1 -5 (负) → 5.  k=4  → [8, 5, 3, -3, 2, -1]
i=2 3 (正) skip
i=3 -3 (负) → 3.  k=3  → [8, 5, 3, 3, 2, -1]
i=4 2 (正) skip
i=5 -1 (负) → 1.  k=2  → [8, 5, 3, 3, 2, 1]
Phase 1 结束, k=2 (偶数), 末尾不翻
sum = 8+5+3+3+2+1 = 22
```

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int largestSumAfterKNegations(vector<int>& nums, int k) {
            sort(nums.begin(), nums.end(), [](int a, int b) {
                return abs(a) > abs(b);                              // |x| 降序
            });
            for (int i = 0; i < (int)nums.size(); i++) {             // Phase 1: 翻负数
                if (nums[i] < 0 && k > 0) {
                    nums[i] *= -1;
                    k--;
                }
                if (k <= 0) break;                                   // 早停
            }
            if (k % 2 == 1) nums[nums.size() - 1] *= -1;             // Phase 2: 余数压末尾
            return accumulate(nums.begin(), nums.end(), 0);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def largestSumAfterKNegations(self, nums: list[int], k: int) -> int:
            # key=abs + reverse=True: 按绝对值降序排
            # lambda 比 C++ comparator 更紧凑
            nums.sort(key=abs, reverse=True)
            for i in range(len(nums)):
                if nums[i] < 0 and k > 0:
                    nums[i] = -nums[i]
                    k -= 1
                if k <= 0:
                    break
            if k % 2 == 1:
                nums[-1] = -nums[-1]                                 # 末尾元素 (= 最小 |x|)
            return sum(nums)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var largestSumAfterKNegations = function(nums, k) {
        nums.sort((a, b) => Math.abs(b) - Math.abs(a));              // |x| 降序: b - a 反向
        for (let i = 0; i < nums.length; i++) {
            if (nums[i] < 0 && k > 0) {
                nums[i] = -nums[i];
                k--;
            }
            if (k <= 0) break;
        }
        if (k % 2 === 1) nums[nums.length - 1] = -nums[nums.length - 1];
        return nums.reduce((a, b) => a + b, 0);                      // reduce 求和, 等价 C++ accumulate
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导.
- **Space**: O(1) (sort in-place, 不算 sort 内部 stack).

## 易错点

- **必须 |x| 降序, 不是升序**: 升序会先翻小的负数, 浪费 k. 用 `abs(a) > abs(b)` 比较器 (C++) 或 `key=abs, reverse=True` (Python).
- **Phase 2 看奇偶, 不要无脑翻**: 偶数次翻等于不翻. 写 `for (i = 0; i < k; i++) nums[last] *= -1` 在 k 大时是 O(k) 浪费, 而且没意义.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同款 sort + 贪心
- [0376. Wiggle Subsequence](../0376-wiggle-subsequence/README.md) — 一遍扫贪心
- 0860\. Lemonade Change (待补) — 贪心找零, 优先大面额
- 0738\. Monotone Increasing Digits (待补) — 贪心改数字
- 1846\. Maximum Element After Decreasing and Rearranging (待补) — sort + 贪心调整

# 0152. Maximum Product Subarray / 乘积最大子数组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Array · 动态规划, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-product-subarray/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给整数数组 `nums`, 返回**乘积最大** 的非空子数组 (连续) 的乘积.

**中文**: 求乘积最大的非空连续子数组的乘积.

## Key Insights

1. **🔑 跟 [0053 Maximum Subarray (求和)](../../09-greedy/0053-maximum-subarray/README.md) 的核心差别: 乘法会翻转符号 / Product flips sign**

    0053 (求和) 只需要追踪"以 i 结尾的最大和" — 加法不会让最值翻转. **但乘法 `负 × 负 = 正`**, 一个超大负数乘上当前负值瞬间变成最大值候选. 所以**必须同时维护"以 i 结尾的最大值 + 最小值"**.

    | | [0053 (求和)](../../09-greedy/0053-maximum-subarray/README.md) | **0152 (求积)** |
    |---|---|---|
    | 运算 | `+` | `×` |
    | 符号翻转? | 否 | **是** (负×负 = 正) |
    | 维护状态 | `curMax` 一个 | **`curMax` + `curMin` 两个** |

    > **见到"乘积/异或/带符号" 类最值** → 立刻想"要不要维护两个反向状态". 这是 0152 留给所有"带翻转" 题的思维印记.

2. **状态: `curMax[i] = 以 i 结尾的最大积`, `curMin[i] = 以 i 结尾的最小积` (强制结尾) / End-forced with two flavors**

    跟 [0300 LIS](../0300-longest-increasing-subsequence/README.md) / [0718](../0718-maximum-length-of-repeated-subarray/README.md) 一样"强制以 i 结尾", 但有两个变种 — 一个最大, 一个最小. 任何当前最大值, 未来都可能因为乘负数变最小; 反之亦然 → 两个都得追踪.

3. **🔑 转移: 三个候选取 max / min / Three candidates**

    每个新元素 `nums[i]` 加入时, 以 i 结尾的最大积有三种可能:

    - **重新起头**: 只用 `nums[i]` 自己 (前面太烂, 断开)
    - **接最大延续**: `curMax_old × nums[i]` (前正后正 / 前负后负)
    - **接最小翻转**: `curMin_old × nums[i]` (前负后负翻成大正)

    $$\begin{aligned}
    curMax_{new} &= \max(nums[i],\ curMax_{old} \times nums[i],\ curMin_{old} \times nums[i]) \\
    curMin_{new} &= \min(nums[i],\ curMax_{old} \times nums[i],\ curMin_{old} \times nums[i])
    \end{aligned}$$

    > **"重新起头" 这个候选** 是 max-subarray 家族共有的 — 跟 [0053 Kadane](../../09-greedy/0053-maximum-subarray/README.md) 一样.

4. **🔑 `tempMax` 必须暂存 / Save the old curMax before updating**

    Yang 写了 `int tempMax = curMax;` 然后再算两个 new — 这是**必需** 的. 直接连写两行会先把 `curMax` 覆盖了, `curMin` 算时拿到的就是**新值**, 错.

    ```cpp
    int tempMax = curMax;                          // ⚠ 先暂存
    curMax = max({nums[i], tempMax * nums[i], curMin * nums[i]});
    curMin = min({nums[i], tempMax * nums[i], curMin * nums[i]});   // ← 这里要用旧 curMax
    ```

    或者用元组解构 (Python / JS) 一次性更新, 避免暂存. C++17 也能 `tie`, 但显式 `temp` 最清楚.

5. **答案 `max(result, curMax)` 全程跟踪 / Track global max throughout**

    跟"以 i 结尾" 一致, 答案在某个 i 结束 → 边算边更新 `result = max(result, curMax)`.

6. **`max({…})` C++ 初始化列表写法 / Multi-arg max via initializer list**

    `max(a, b)` 只接两参. 想比三个: `max({a, b, c})` (花括号 = `initializer_list`). C++11+ 支持. Yang 用了这个, 比 `max(a, max(b, c))` 干净.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxProduct(vector<int>& nums) {
            int curMax = nums[0], curMin = nums[0], result = nums[0];
            for (int i = 1; i < (int)nums.size(); i++) {
                int tempMax = curMax;                              // ⚠ 暂存旧 curMax
                curMax = max({nums[i], tempMax * nums[i], curMin * nums[i]});
                curMin = min({nums[i], tempMax * nums[i], curMin * nums[i]});
                result = max(result, curMax);
            }
            return result;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxProduct(self, nums: list[int]) -> int:
            # Python 元组解构: 右边先全部求值再赋给左边 — 不用 tempMax
            # 等价 C++ 的"先 tempMax 暂存" 写法, 但更简洁
            cur_max = cur_min = result = nums[0]
            for x in nums[1:]:
                cur_max, cur_min = (
                    max(x, cur_max * x, cur_min * x),
                    min(x, cur_max * x, cur_min * x),
                )
                result = max(result, cur_max)
            return result
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var maxProduct = function(nums) {
        let curMax = nums[0], curMin = nums[0], result = nums[0];
        for (let i = 1; i < nums.length; i++) {
            // 解构 + 右侧表达式提前求值, 模拟 Python 元组
            const tempMax = curMax;
            curMax = Math.max(nums[i], tempMax * nums[i], curMin * nums[i]);
            curMin = Math.min(nums[i], tempMax * nums[i], curMin * nums[i]);
            result = Math.max(result, curMax);
        }
        return result;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 相关题目

- [0053. Maximum Subarray](../../09-greedy/0053-maximum-subarray/README.md) — **求和版**, 不需要维护 min — 加法不会翻符号
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — 同款"以 i 结尾" 状态
- [0718. Maximum Length of Repeated Subarray](../0718-maximum-length-of-repeated-subarray/README.md) — 同款"子数组 + 强制结尾"
- 1567\. Maximum Length of Subarray With Positive Product (待补) — 直接套本题思路, 长度版
- 0918\. Maximum Sum Circular Subarray (待补) — 0053 环形版 + "拆环为链" 思路
- 0628\. Maximum Product of Three Numbers (待补) — 同款"乘法要看 min" 思维, 但三个数固定

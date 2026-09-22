# 0918. Maximum Sum Circular Subarray / 环形子数组的最大和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Greedy, Kadane, DP · 数组, 贪心, Kadane, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-sum-circular-subarray/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Two cases**: the max subarray is either **non-circular** (regular Kadane on `nums`) or **circular** (equals `total - minSubarray`, since removing the min-sum contiguous chunk leaves the max wraparound piece). Answer = `max(maxSum, total - minSum)`. **Edge**: if all nums are negative, `total - minSum = 0` (empty) — return `maxSum` instead.
>
> **中文**: **两种情况**: 最大子数组要么**不跨环** (普通 Kadane) 要么**跨环** = `总和 - 最小子数组` (挖掉最小连续段, 剩下正好是环上跨界最大段). 答案 = `max(maxSum, total - minSum)`. **边界**: 全负时 `total - minSum = 0` (空), 返 maxSum.
>
> *Template / 模版*: **Kadane × 2 + complement trick** — 环形数组类问题的通用招.

## Problem

**EN**: 给环形数组 `nums`, 找**任一连续段** (可跨末首) 的最大和. 段非空.

**中文**: 环形连续子数组最大和, 段非空.

**Example**: `nums = [5, -3, 5]` → 环形 `[5, 5]` 跨越末首, 和 = 10.

## Key Insights

1. **🔑 灵魂: 两种情况穷尽 / Two cases exhaustively**

    环形数组最大子数组只可能是这两种形态之一:

    ```
    Case 1: 不跨环
    [ ... [maxSubarray] ... ]           ← Kadane 直接得
    
    Case 2: 跨环 (wraparound)
    [maxSubarray末] ... [minSubarray] ... [maxSubarray首]
                        ^^^^^^^^^^^^ 挖掉这段
    ```

    → 答案 = **max(Case 1, Case 2)**. Case 2 值 = **total - minSubarray**.

    > **"跨环 ≡ 补集是最小子数组"** — 关键洞察. 挖掉最小连续段, 剩下的正好是"两头"合起来的**跨环段**.

2. **🔑 灵魂: 一次遍历同时跑两个 Kadane / One-pass dual Kadane**

    Yang 代码在**一次遍历里同时维护**:

    - `curMax / maxSum`: 标准 Kadane (max 子数组和).
    - `curMin / minSum`: 镜像 Kadane (min 子数组和).
    - `total`: 数组总和.

    ```cpp
    curMax = max(curMax + x, x);      // Kadane 转移: 接或重启
    maxSum = max(maxSum, curMax);
    curMin = min(curMin + x, x);      // 镜像: min 版
    minSum = min(minSum, curMin);
    ```

    → 时间 O(n), 空间 O(1).

3. **🔑 灵魂: 全负边界 — `total - minSum = 0` 是空子数组 / All-negative edge case**

    若所有元素**都负**:

    - `minSum = total` (整个数组本身就是最小).
    - `total - minSum = 0` → 对应**空子数组** (挖掉整个数组).
    - 但题目要求**非空段**! → 不能返 0.

    此时 `maxSum` 是**最大的那个负数** (单元素段), 应返 maxSum.

    Yang 的判据: **`maxSum > 0`**:

    - `maxSum > 0`: 至少有一个正段, 环形答案取 `max(maxSum, total - minSum)`.
    - `maxSum ≤ 0`: 全负 (或全零), 只能返 maxSum.

    > **易错点 top 1**: 忘处理全负边界, `total - minSum = 0` 会被误选. `if (maxSum > 0)` 一行救命.

4. **🔑 为啥"挖掉 minSubarray" 等价"取跨环 maxSubarray" / Why complement works**

    数学: 数组 `A` 上任意连续段 `[l, r]`, 它的**补集** = `A[0..l-1] + A[r+1..n-1]` = 跨环的两段.

    - `sum(补集) = total - sum([l, r])`.
    - 补集是**跨环连续段** (环上首尾拼起来).

    → 想让**跨环段最大**, 等价**挖掉的段 (`[l, r]`) 最小** → **min subarray**.

    > **"补集恒等" 是环形转线性的通用桥梁**. 也见环形字符串, 环形游戏等.

5. **🔑 curMax / curMin 初始 0 vs nums[0] / Initialization detail**

    Yang 用:

    ```cpp
    int curMax = 0, maxSum = nums[0];
    int curMin = 0, minSum = nums[0];
    ```

    - **`curMax = 0`**: 空段, 遍历第一个 x 时 `max(0+x, x) = x` — 相当于从 x 重启. **合法**.
    - **`maxSum = nums[0]`**: 保证至少有 1 元素 (非空段).
    - **对称**: curMin/minSum 同理.

    若 `maxSum = 0` 初始, 全负数组会返 0 (空段), **错**. 必须 `nums[0]`.

    > **易错点 top 2**: max/minSum 初始必须 nums[0], 不能 0. 否则全负时错.

6. **🔑 复杂度 / Complexity**

    - **Time**: O(n) — 一遍扫两个 Kadane 并行.
    - **Space**: O(1).

7. **🔑 替代解法 (备用) / Alternative approaches**

    | 方法 | Time | Space | 特点 |
    |---|---|---|---|
    | **双 Kadane** (Yang) | **O(n)** | **O(1)** | 最优 |
    | 数组翻倍 + Kadane | O(n) | O(n) | 拼两遍 `nums+nums`, 加长度约束扫 |
    | 单调 deque + 前缀和 | O(n) | O(n) | 通用 |

    双 Kadane 是**最简洁**且 O(1) 空间.

8. **🔑 相关: 0053 母题 / Related to 0053**

    - **0053 Maximum Subarray** ([§09](../0053-maximum-subarray/README.md)): 线性版, 单 Kadane.
    - **0918** (本题): 环形版, **双 Kadane + 补集招**.

    > 见过 0053 → 本题只需"多加一步 minSum + total-minSum" 即可. **模式复用最省心**.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxSubarraySumCircular(vector<int>& nums) {
            int total  = 0;
            int curMax = 0, maxSum = nums[0];       // ⚠️ maxSum 初始 nums[0] 不是 0
            int curMin = 0, minSum = nums[0];

            for (int x : nums) {
                total += x;
                curMax = max(curMax + x, x);        // Kadane: 接前 or 重启
                maxSum = max(maxSum, curMax);
                curMin = min(curMin + x, x);        // 镜像 Kadane
                minSum = min(minSum, curMin);
            }

            // 全负边界: maxSum > 0 才考虑跨环补集
            return maxSum > 0 ? max(maxSum, total - minSum) : maxSum;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxSubarraySumCircular(self, nums: list[int]) -> int:
            # 双 Kadane 一次扫; Python 没 max/min 的原地版, 每步都覆盖
            total = 0
            cur_max, max_sum = 0, nums[0]           # max_sum 必须 nums[0], 不能 0
            cur_min, min_sum = 0, nums[0]

            for x in nums:
                total += x
                # Python max/min 支持多参数, 相当于 C++ 的 std::max
                cur_max = max(cur_max + x, x)
                max_sum = max(max_sum, cur_max)
                cur_min = min(cur_min + x, x)
                min_sum = min(min_sum, cur_min)

            # 三元表达式: Python 写法 A if cond else B
            # 相当于 C++ 的 cond ? A : B
            return max(max_sum, total - min_sum) if max_sum > 0 else max_sum
    ```

=== "JavaScript"
    ```javascript
    var maxSubarraySumCircular = function(nums) {
        // Math.max/min 支持多参数; 每步覆盖变量
        let total = 0;
        let curMax = 0, maxSum = nums[0];
        let curMin = 0, minSum = nums[0];

        for (const x of nums) {
            total += x;
            curMax = Math.max(curMax + x, x);       // Kadane
            maxSum = Math.max(maxSum, curMax);
            curMin = Math.min(curMin + x, x);
            minSum = Math.min(minSum, curMin);
        }

        // 三元 ? : 短语法
        return maxSum > 0 ? Math.max(maxSum, total - minSum) : maxSum;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 相关题目

- [0053. Maximum Subarray](../0053-maximum-subarray/README.md) — 线性 Kadane 母题
- 0152\. Maximum Product Subarray (待补) — Kadane 变形, max/min 双状态
- [0560. Subarray Sum Equals K](../../01-array/0560-subarray-sum-equals-k/README.md) — 前缀和 + hash
- [0523. Continuous Subarray Sum](../../01-array/0523-continuous-subarray-sum/README.md) — 前缀和 + mod
- 0974\. Subarray Sums Divisible by K (待补) — 已上 §01
- 1191\. K-Concat Maximum Sum (待补) — 环形/翻倍变体
- 2321\. Maximum Score of Spliced Array (待补) — 双数组差分 + Kadane
- 1749\. Maximum Absolute Sum of Any Subarray (待补) — max(maxSum, -minSum)

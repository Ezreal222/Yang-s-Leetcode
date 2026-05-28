# 0907. Sum of Subarray Minimums / 子数组的最小值之和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Contribution · 单调栈, 贡献法
    - **Link**: [LeetCode](https://leetcode.com/problems/sum-of-subarray-minimums/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给数组 `arr`. 求**所有连续子数组的最小值之和**, 模 `10^9 + 7`.

**中文**: 求所有子数组的 min 之和, 模 1e9+7.

## Key Insights

1. **🔑 换视角: 每个元素当了多少个子数组的最小值 / Contribution approach**

    朴素枚举子数组 O(n²) × O(n) = O(n³), TLE.

    **换问法**: 不数子数组, 数**每个 `arr[i]` 当过多少次最小值** (记 `cnt[i]`). 答案 = `Σ arr[i] × cnt[i]`.

    > **贡献法 (Contribution Counting)** 是计数类题的核心思维: 与其遍历"包" 数"球", 不如让每个"球" 报出"自己在多少包里出现".

2. **🔑 `cnt[i]` 推导: 左右严格更小的边界 / Bounds via strictly-smaller neighbors**

    `arr[i]` 当子数组 `[L, R]` 的最小值需要:

    - `L ≤ i ≤ R`
    - `arr[L..R]` 中没有比 `arr[i]` 更小的

    设:
    - `left[i]` = 左边第一个 `< arr[i]` 的位置 (无则 -1)
    - `right[i]` = 右边第一个 `< arr[i]` 的位置 (无则 n)

    则 `L ∈ (left[i], i]`, `R ∈ [i, right[i])`. 乘法原理:

    $$cnt[i] = (i - left[i]) \times (right[i] - i)$$

    > **左右独立选, 乘起来**. 这是"贡献法 + 单调栈" 的标准公式.

3. **🔑 ⚠ 平局处理 (tie-breaking): 一侧严格 `<`, 另一侧 `≤` / Avoid double-counting equal elements**

    若 `arr` 有相同值, "谁当最小" 会重复. 经典做法: **一侧用严格 `<`, 另一侧用 `≤`**, 让每个相同值的子数组**唯一归属**给"最左 (或最右) 那一个".

    Yang 的写法 `arr[j] <= arr[stk.top()]`: 来一个等值就弹栈 → 弹出的那个 i 的 right 边界算到当前 j (因为 j 是它的"右边第一个 ≤"). 而左侧因为栈本身是严格递增 (用 `<`), left 是"严格更小". 这样:

    - 弹 i 时: left 严格更小, right 第一个 ≤. 相同值的归属落到**最右那个**.
    - 没有重复 ✓

    > 反向也对: 左 `≤`, 右 `<` → 归属给最左. 选一种**保持一致** 即可. 弄反就重复 / 漏算.

4. **🔑 末尾哨兵: `j == n` 强制清空栈 / Sentinel j == n forces final flush**

    Yang 把循环写成 `for j ≤ n`, 当 `j == n` 时**虚拟一个"比所有都小"** 的元素, 让 while 条件无条件触发 — 弹出栈里剩下的所有索引, 各自结算 right = n.

    没这个哨兵, 单调栈结束后栈里还有元素 → 它们的 right 没确定. 哨兵优雅地把"清栈" 融入主循环.

    > 跟 [0084](../0084-largest-rectangle-in-histogram/README.md) 末尾 `push_back(0)` 哨兵同精神. 后者是改数组, 这里是 `j == n` 条件触发.

5. **复杂度 O(n) 摊销 / Amortized O(n)**

    每个索引入栈一次, 出栈一次 → 主循环总 O(n). 比朴素 O(n³) 快两个量级.

6. **本题 = [1504](../1504-count-submatrices-with-all-ones/README.md) 优化的核心子问题 / Building block for 1504 O(m·n)**

    [1504 统计全 1 子矩形](../1504-count-submatrices-with-all-ones/README.md) v2 的内层就是"对每行 heights 跑本题模板". 学透 0907, 1504 的 O(m·n) 自动会写.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int sumSubarrayMins(vector<int>& arr) {
            const int MOD = 1e9 + 7;
            int n = arr.size();
            stack<int> stk;                                        // 栈存索引, 单调严格递增
            long ans = 0;

            for (int j = 0; j <= n; j++) {                         // j == n 是哨兵, 强制清栈
                // ⚠ <= 让平局右侧弹出 (避免双算)
                while (!stk.empty() && (j == n || arr[j] <= arr[stk.top()])) {
                    int i = stk.top(); stk.pop();
                    int leftBound = stk.empty() ? -1 : stk.top();  // 左边第一个 < arr[i] (严格)
                    int rightBound = j;                            // 右边第一个 ≤ arr[i]
                    long cnt = (long)(i - leftBound) * (rightBound - i);
                    ans = (ans + arr[i] * cnt) % MOD;
                }
                stk.push(j);
            }
            return (int)ans;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sumSubarrayMins(self, arr: list[int]) -> int:
            MOD = 10**9 + 7
            n = len(arr)
            stk = []
            ans = 0

            # j 走到 n 当哨兵, 强制清空栈
            for j in range(n + 1):
                while stk and (j == n or arr[j] <= arr[stk[-1]]):
                    i = stk.pop()
                    left_bound = stk[-1] if stk else -1
                    right_bound = j
                    cnt = (i - left_bound) * (right_bound - i)
                    ans = (ans + arr[i] * cnt) % MOD
                stk.append(j)
            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} arr
     * @return {number}
     */
    var sumSubarrayMins = function(arr) {
        const MOD = 1_000_000_007n;                                // BigInt 防中间溢出
        const n = arr.length;
        const stk = [];
        let ans = 0n;

        for (let j = 0; j <= n; j++) {
            while (stk.length && (j === n || arr[j] <= arr[stk[stk.length - 1]])) {
                const i = stk.pop();
                const leftBound = stk.length ? stk[stk.length - 1] : -1;
                const cnt = BigInt((i - leftBound) * (j - i));
                ans = (ans + BigInt(arr[i]) * cnt) % MOD;
            }
            stk.push(j);
        }
        return Number(ans);
    };
    ```

## Complexity

- **Time**: O(n) — 每索引入栈出栈各一次.
- **Space**: O(n) — 栈.

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 单调栈基础
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 模板
- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 哨兵 + 单调栈同套路
- [1504. Count Submatrices With All Ones](../1504-count-submatrices-with-all-ones/README.md) — 本题的二维应用 (每行跑一次)
- 1856\. Maximum Subarray Min-Product (待补) — 同款贡献法 + 单调栈
- 2104\. Sum of Subarray Ranges (待补) — `Σ (max - min)`, 跑两遍单调栈

# 3954. Sum of Compatible Numbers in Range I / 范围内兼容数字的和 I

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Bit Manipulation, Math · 位运算, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/sum-of-compatible-numbers-in-range-i/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给整数 `n` 和 `k`. 正整数 `x` 称为**兼容** 当且仅当:

1. `abs(n - x) ≤ k`
2. `(n & x) == 0` (位 AND 为 0, 即**无共同二进制位**)

返回所有兼容 x 的和.

**中文**: 找所有满足"跟 n 差 ≤ k 且没共同二进制位" 的正整数 x, 求和.

## Key Insights

1. **🔑 两条件分开理解 / Decompose two conditions**

    - **范围**: `abs(n - x) ≤ k` ⟺ `x ∈ [n - k, n + k]`. 加上正整数约束 → `x ∈ [max(1, n - k), n + k]`.
    - **位无交**: `(n & x) == 0` ⟺ n 和 x **没有共同的 1 位**. 等价于 x 是 n 按位取反 (在 n 的二进制位上)的某种"位补集" 子集.

    > **任何位运算条件都先翻译成"位的语义"**, 别被符号吓到. `& == 0` 就是"无重叠 1 位".

2. **🔑 数据规模决定: 暴力扫即可 / Brute force scan suffices**

    `n ≤ 100`, `k ≤ 100` → 区间 `[max(1, n-k), n+k]` 最多 201 个数. 直接 for 循环每个判 `(n & x) == 0`. **O(k) 时间, O(1) 空间**.

    > 看到极小数据规模 (n ≤ 100) → 别想复杂. 朴素 O(k) 直接过.

3. **`max(1, n - k)` 兜底 / Lower bound guard**

    若 `n - k ≤ 0`, 起点要从 1 开始 (题目要求 x 是**正整数**). 漏写 `max(1, ...)` 会从 0 或负数开始 — 虽然 `0 & n == 0`, 但 0 不是正整数, 加进答案就 WA.

    > **题目约束"正整数"** 容易漏看, `x > 0` 必须保证.

4. **`(n & x) == 0` 的含义 / Bit-disjoint condition**

    举例 `n = 2 = 0b10`. x 兼容 ⟺ x 二进制里**不能有第 1 位** (n 用了的位).

    - `x = 1 = 0b01`: ✓
    - `x = 4 = 0b100`: ✓
    - `x = 5 = 0b101`: ✓
    - `x = 3 = 0b11`: ✗ (含第 1 位)
    - `x = 6 = 0b110`: ✗

    > 想象成"n 占了某些抽屉, x 必须用别的抽屉". 跟"子集枚举" 的位运算思维相通.

5. **复杂度 O(k) / Linear in k**

    扫一遍 `n - k` 到 `n + k`, 每次 O(1) 位 AND 检查. 总 O(2k + 1) = O(k).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int sumOfGoodIntegers(int n, int k) {
            int sum = 0;
            int lo = max(1, n - k);                                 // 保证正整数
            int hi = n + k;
            for (int x = lo; x <= hi; x++) {
                if ((n & x) == 0) sum += x;                         // 位无交即兼容
            }
            return sum;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sumOfGoodIntegers(self, n: int, k: int) -> int:
            # sum + 生成器一行算: 区间扫 + & 检查
            return sum(x for x in range(max(1, n - k), n + k + 1) if (n & x) == 0)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @param {number} k
     * @return {number}
     */
    var sumOfGoodIntegers = function(n, k) {
        let sum = 0;
        for (let x = Math.max(1, n - k); x <= n + k; x++) {
            if ((n & x) === 0) sum += x;
        }
        return sum;
    };
    ```

## Complexity

- **Time**: O(k) — 区间 `[max(1, n-k), n+k]` 最多 2k+1 个数.
- **Space**: O(1).

## 相关题目

- 0136\. Single Number (待补) — 位 XOR 基础
- 0137\. Single Number II (待补) — 位计数进阶
- 0260\. Single Number III (待补) — 位 XOR + 分组
- 0078\. Subsets (待补) — 用 bitmask 枚举所有子集
- 0405\. Convert a Number to Hexadecimal (待补) — 位操作 + 进制
- 0461\. Hamming Distance (待补) — XOR + popcount
- 0191\. Number of 1 Bits (待补) — popcount 基础

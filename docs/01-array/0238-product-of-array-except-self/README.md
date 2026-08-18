# 0238. Product of Array Except Self / 除自身以外数组的乘积

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Prefix Product · 数组, 前缀积
    - **Link**: [LeetCode](https://leetcode.com/problems/product-of-array-except-self/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **No division allowed** (zeros) → **left prefix product × right suffix product**. v2: reuse `ans` for the left pass, then sweep right with one scalar → O(1) extra space.
>
> **中文**: **不准除** (有 0) → **左前缀积 × 右后缀积**. v2 把 ans 当左积载体, 右往左扫加一个 scalar — O(1) 额外空间.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给数组 `nums`. 返回 `ans[i] = 所有 nums[j] (j ≠ i) 的乘积`. 不准用除法, O(n) 时间. Follow-up: O(1) 额外空间 (output 不算).

**中文**: 除自己外的乘积. 禁除法.

## Key Insights

1. **🔑 关键约束: 禁除法 → 必须双向扫 / Division banned ⇒ two-pass scan**

    若可以除: `ans[i] = total / nums[i]`, O(n) 解决. 但 **nums 可能含 0** — 除 0 爆炸; 多个 0 时谁该是 0 也乱.

    **禁除法之后, "除自身" 等价于"左边的积 × 右边的积"**:

    ```
    ans[i] = (nums[0] × ... × nums[i-1]) × (nums[i+1] × ... × nums[n-1])
              ←——— 左前缀积 ———→         ←——— 右后缀积 ———→
    ```

    > **"无法局部计算" 时, 拆成"双向预处理"** — 跟前缀和 [0303](../0303-range-sum-query-immutable/README.md) 一脉相承, 把"乘" 换成"加" 就是同一思想.

2. **🔑 边界用单位元 1 (乘法 identity) / Boundary = 1 (multiplicative identity)**

    `left[0]` 表示"0 号位置左边的积" — 左边**没有元素**, 用乘法单位元 **1** 兜底. 同理 `right[n-1] = 1`.

    > 加法的单位元是 0 (前缀和的 sum[0] = 0), 乘法的单位元是 1. **同款思想换运算符**.

3. **🔑 v1 → v2 空间优化: ans 复用 + scalar / Reuse ans + scalar**

    v1: `left[]` + `right[]` + `ans[]` 三个数组, **O(n) 额外**.

    v2 (Yang 巧妙 trick):

    - **第一遍**: `ans[i]` 先存"左前缀积" (复用输出数组当临时空间).
    - **第二遍**: 从右往左扫, 维护一个 scalar `right` 累积右后缀积, 直接乘进 `ans[i]`.

    ```cpp
    ans[i] = ans[i] (已是左积) * right (累积的右积)
    right *= nums[i]                                    // 推进右积
    ```

    > **"输出数组本来就要分配, 不算 extra space" 是题目允许的小聪明**. v2 是 Follow-up 标准答案.

4. **🔑 为啥 ans 初始化为 1 而不是 0? / Why init ans to 1**

    v2 中, **第一格 `ans[0]` 直接代表"0 号位置左边的积"** = 1 (左边没东西). 后续 `ans[i] = ans[i-1] * nums[i-1]` 才能正确递推.

    若初始化为 0, 第一步就乘成 0, 全数组爆炸.

5. **🔑 双向扫的两种推进方式 / Two ways to advance**

    | | v1 (left + right 数组) | v2 (ans + scalar) |
    |---|---|---|
    | 左积递推 | `left[i] = left[i-1] * nums[i-1]` | `ans[i] = ans[i-1] * nums[i-1]` (同公式) |
    | 右积载体 | `right[]` 数组 | 单个 `int right = 1` |
    | 合并 | 单独一遍 `ans[i] = left[i] * right[i]` | **合在右扫里** `ans[i] *= right; right *= nums[i]` |
    | extra space | O(n) | **O(1)** |

    > v1 思路最清晰, **教学先讲**. v2 是工程版, **面试 Follow-up 必备**.

6. **复杂度 O(n) 时间, O(1) 额外空间 (v2) / Linear time, constant extra**

    两遍线性扫. 输出数组不算 extra (题目允许).

## Solution

=== "C++"

    **v1: 显式 left + right 数组 (思路最清晰)**

    ```cpp
    class Solution {
    public:
        vector<int> productExceptSelf(vector<int>& nums) {
            int n = nums.size();
            vector<int> left(n, 1), right(n, 1), ans(n);
            for (int i = 1; i < n; i++) left[i] = left[i - 1] * nums[i - 1];       // 左前缀积
            for (int i = n - 2; i >= 0; i--) right[i] = right[i + 1] * nums[i + 1]; // 右后缀积
            for (int i = 0; i < n; i++) ans[i] = left[i] * right[i];
            return ans;
        }
    };
    ```

    **v2: ans 复用 + scalar (O(1) 额外空间)**

    ```cpp
    class Solution {
    public:
        vector<int> productExceptSelf(vector<int>& nums) {
            int n = nums.size();
            vector<int> ans(n, 1);
            // 第一遍: ans[i] 存"i 左边的积"
            for (int i = 1; i < n; i++) ans[i] = ans[i - 1] * nums[i - 1];
            // 第二遍: right 累积"i 右边的积", 直接乘进 ans[i]
            int right = 1;
            for (int i = n - 1; i >= 0; i--) {
                ans[i] *= right;
                right *= nums[i];
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def productExceptSelf(self, nums: list[int]) -> list[int]:
            # v2: O(1) extra space. 直接对应 C++ v2
            # 注意 Python 没有 vector<int>(n, 1) 这种语法, 用列表乘法 [1] * n
            n = len(nums)
            ans = [1] * n
            # 第一遍: 左前缀积
            for i in range(1, n):
                ans[i] = ans[i - 1] * nums[i - 1]
            # 第二遍: 右后缀积用 scalar 累积. enumerate 不需要, 反向 range 更直观
            right = 1
            for i in range(n - 1, -1, -1):
                ans[i] *= right
                right *= nums[i]
            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number[]}
     */
    var productExceptSelf = function(nums) {
        // v2: O(1) extra space
        const n = nums.length;
        // new Array(n).fill(1) — 基本类型 fill 没"共享引用" 坑, 这里安全
        const ans = new Array(n).fill(1);
        // 第一遍: 左前缀积
        for (let i = 1; i < n; i++) ans[i] = ans[i - 1] * nums[i - 1];
        // 第二遍: 右扫 + scalar
        let right = 1;
        for (let i = n - 1; i >= 0; i--) {
            ans[i] *= right;
            right *= nums[i];
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(n) — 两遍线性扫.
- **Space**: O(1) extra (v2) — 不含输出数组. v1 是 O(n).

## 相关题目

- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 前缀**和**, 同款"区间运算前置预处理"
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维前缀和
- 0152\. Maximum Product Subarray (待补) — 乘积版"最大子数组", 需处理负号翻转
- 0042\. Trapping Rain Water (待补 - 已存) — 双向预处理母题, 左右最大值替换"积"
- [0560. Subarray Sum Equals K](../0560-subarray-sum-equals-k/README.md) — 前缀和 + 哈希
- 1352\. Product of the Last K Numbers (待补) — 流式前缀积

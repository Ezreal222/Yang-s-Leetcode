# 0303. Range Sum Query - Immutable / 区域和检索 - 数组不可变

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Array, Prefix Sum, Design · 数组, 前缀和, 设计
    - **Link**: [LeetCode](https://leetcode.com/problems/range-sum-query-immutable/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Design a structure for static array `nums` that supports `sumRange(left, right)` returning sum of `nums[left..right]` (inclusive). Many queries, the array does **not** change.

**中文**: 静态数组 `nums`, 设计结构支持 `sumRange(left, right)`, 返回 `nums[left..right]` 闭区间和. 多次查询, 数组不变.

## Key Insights

1. **前缀和模板 / Prefix sum template**

    `prefix[i] = nums[0] + nums[1] + ... + nums[i-1]` (前 i 项和, **不含 i**).

    则 `sum(nums[l..r]) = prefix[r+1] - prefix[l]`. 一次 O(n) 预处理, 之后每次查询 O(1).

    > 朴素做法每次查询 O(n) 累加 → 多次查询 O(n × q). 前缀和把"重复算" 提前完成, 查询降到 O(1).

2. **哨兵 (sentinel) 让边界统一 / Sentinel `prefix[0] = 0` removes corner case**

    把 `prefix` 大小开成 `n + 1`, `prefix[0] = 0`, `prefix[i+1] = prefix[i] + nums[i]`.

    这样 `sumRange(l, r) = prefix[r+1] - prefix[l]` **不用对 `l == 0` 特判**. 哨兵代价: 多 1 个 int.

    > 没哨兵: `sum = prefix[r] - (l == 0 ? 0 : prefix[l-1])` — 丑且易错.

3. **闭区间公式 vs 半开区间公式 / Closed vs half-open formulas**

    | 习惯 | prefix 定义 | sumRange(l, r) |
    |---|---|---|
    | **本题 (闭区间 + 哨兵)** | `prefix[i] = nums[0..i-1] 之和` | `prefix[r+1] - prefix[l]` |
    | C++ STL 习惯 (`partial_sum`) | 同上 | 同上 (`partial_sum` 写入从下标 1 开始) |
    | 简化版 (无哨兵) | `prefix[i] = nums[0..i] 之和` | `prefix[r] - prefix[l-1]` (l=0 特判) |

    > 一句话: **哨兵 + "前 i 项 (不含 i)"** 的定义最不容易写错, 跟 Python `itertools.accumulate(..., initial=0)` 同源.

4. **跟差分数组的对偶 / Prefix sum vs diff array (dual)**

    - **前缀和**: 多次"区间和查询", 数组不变 → **预处理一次, O(1) 查**.
    - **差分**: 多次"区间整体加, 最后查每点" → **O(1) 改, 最后一次 prefix sum**.

    两者数学上互为逆运算 (`diff[i] = nums[i] - nums[i-1]`). 看到"批量区间操作" 就在两者间挑.

5. **进阶变体 / Family extensions**

    - 0307\. Range Sum Mutable (待补) → 数组会变 → 前缀和重算太贵 → **树状数组 / 线段树**, O(log n) 查 + 改.
    - 0304 Range Sum 2D (待补) → 二维前缀和, `prefix[i][j] = 矩形 (0,0)-(i-1,j-1) 之和`.
    - 0560 Subarray Sum Equals K (待补) → 前缀和 + 哈希表反查.

## Solution

=== "C++"
    ```cpp
    class NumArray {
        vector<int> prefix;                                        // prefix[i] = nums[0..i-1] 之和 (哨兵)
    public:
        NumArray(vector<int>& nums) {
            int n = nums.size();
            prefix.assign(n + 1, 0);                               // prefix[0] = 0 哨兵, 其余先填 0
            for (int i = 0; i < n; i++) {
                prefix[i + 1] = prefix[i] + nums[i];               // 递推一行
            }
        }
        int sumRange(int left, int right) {
            return prefix[right + 1] - prefix[left];               // 闭区间和 = prefix[r+1] - prefix[l]
        }
    };
    ```

=== "Python"
    ```python
    from itertools import accumulate

    class NumArray:
        def __init__(self, nums: list[int]):
            # accumulate(nums, initial=0): 一行做前缀和 + 哨兵
            # initial=0 等价 C++ 手动写 prefix[0] = 0; 返回长度 n+1 的迭代器
            # 等价 for 循环: prefix = [0]; for x in nums: prefix.append(prefix[-1] + x)
            self.prefix = list(accumulate(nums, initial=0))

        def sumRange(self, left: int, right: int) -> int:
            return self.prefix[right + 1] - self.prefix[left]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     */
    var NumArray = function(nums) {
        // 一开就 length+1, 第 0 位天然 0 (JS new Array(n) 配合 fill 是 undefined, 这里直接累加更稳)
        this.prefix = new Array(nums.length + 1).fill(0);
        for (let i = 0; i < nums.length; i++) {
            this.prefix[i + 1] = this.prefix[i] + nums[i];
        }
    };

    /**
     * @param {number} left
     * @param {number} right
     * @return {number}
     */
    NumArray.prototype.sumRange = function(left, right) {
        return this.prefix[right + 1] - this.prefix[left];
    };
    ```

## Complexity

- **Time**: 构造 O(n); 每次 `sumRange` O(1).
- **Space**: O(n) — prefix 数组.

## 相关题目

- [1094. Car Pooling](../../09-greedy/1094-car-pooling/README.md) — 同族的对偶: 差分数组 + 前缀和验容量
- [0304. Range Sum Query 2D - Immutable](../0304-range-sum-query-2d-immutable/README.md) — 二维版, 容斥原理 + 哨兵
- 0307\. Range Sum Query - Mutable (待补) — 数组可变 → 树状数组 / 线段树
- 0560\. Subarray Sum Equals K (待补) — 前缀和 + 哈希
- 0724\. Find Pivot Index (待补) — 前缀和入门变种
- 0974\. Subarray Sums Divisible by K (待补) — 前缀和 + 余数哈希

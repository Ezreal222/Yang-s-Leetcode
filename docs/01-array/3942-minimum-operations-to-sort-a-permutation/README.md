# 3942. Minimum Operations to Sort a Permutation / 排序排列的最少操作

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Math, Permutation, Greedy · 数组, 数学, 排列, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-operations-to-sort-a-permutation/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 `[0, n-1]` 的排列 `nums`. 允许两种操作:

1. **反转** 整个数组
2. **左移 1**: 把首元素放到末尾, 其他左移一位

求把 `nums` 排成升序的最少操作数, 不可能则返回 `-1`.

**中文**: 排列上做"反转 + 左移 1" 操作, 求最少次数排成升序, 不行返 `-1`.

## Key Insights

1. **🔑 可达集 = 2n 个特殊置换 / Reachable set is exactly 2n permutations**

    从已排序 `[0..n-1]` 出发, 任意 L (左移) + R (反转) 序列只能到达:

    - **n 个升序循环移位**: `[k, k+1, ..., n-1, 0, ..., k-1]`, k = 0..n-1
    - **n 个降序循环移位**: `[k, k-1, ..., 0, n-1, ..., k+1]`, k = 0..n-1

    一共 **2n 个**. 其他排列都不可达 → 直接返 `-1`.

    > 这是题目最重要的观察 — 操作群的轨道大小决定可行性. 不在轨道里, 怎么操作都到不了.

2. **🔑 可行性判断: 数循环相邻"破" 的个数 / Feasibility = count cyclic breaks**

    对 `i` 的下一位置 `nxt = (i+1) % n`:

    - **`up` 计数**: 满足 `(nums[i] + 1) % n != nums[nxt]` 的对数 — **升序循环违规数**
    - **`down` 计数**: 满足 `(nums[i] - 1 + n) % n != nums[nxt]` 的对数 — **降序循环违规数**

    | 情况 | 含义 |
    |---|---|
    | `up == 0` | 完美升序循环 → 可达 |
    | `down == 0` | 完美降序循环 → 可达 |
    | 都 > 0 | 既非升也非降 → **不可达, 返 -1** |

3. **`up == 1` 不可能 / Exactly 1 break is impossible — `<= 1` is loose but harmless**

    Yang 写的是 `up <= 1 || down <= 1`. 严格来说应该是 `== 0`. 但**置换不可能恰好有 1 个 break**:

    若 n-1 对相邻满足 `+1 mod n`, 那么走过整圈正好回到起点 ⟹ 闭合那条边也必然满足 ⟹ up = 0.

    所以 `up` 只能取 `0` 或 `≥ 2`. Yang 的 `<= 1` 跟 `== 0` 等价, 不出错.

4. **🔑 排序代价: `min(maxIdx + 1, n - maxIdx + 1)` / Cost formula**

    设 `maxIdx` = 最大值 `n-1` 的位置. 两个对称策略:

    | 策略 | 升序循环移位 | 降序循环移位 | 代价 |
    |---|---|---|---|
    | **A** | `maxIdx + 1` 次左移 (把 0 转到首位) | `maxIdx` 次左移 + 1 次反转 (把 max 转到首位再翻) | `maxIdx + 1` |
    | **B** | 1 反转 + `n-1-maxIdx` 左移 + 1 反转 | 1 反转 + `n - maxIdx` 左移 (转为升序后再排) | `n - maxIdx + 1` |

    取较小 ⟹ `min(maxIdx + 1, n - maxIdx + 1)`. 两种 cyclic 都用**同一公式**, 路径不同.

5. **特判: 已经排好 → 返 0 / Sorted early-return**

    若 `nums = [0, 1, ..., n-1]`, 是升序循环移位的 `k=0` 特例. 此时 `maxIdx = n-1`, 公式给 `min(n, 2) = 2` — **不对**.

    要单独判 `sorted`, 直接返 `0`. 公式假设至少做一步操作, 已排好不需要做.

6. **n=1 特判 / Degenerate single element**

    n=1 时数组只能是 `[0]`, 已是排序. Yang 在 `test` 开头 `if (n == 1) return true` 短路, 主函数里 `sorted = true` 再返 0.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool test(const vector<int>& nums) {
            int n = nums.size();
            if (n == 1) return true;
            int up = 0, down = 0;
            for (int i = 0; i < n; i++) {
                int nxt = (i + 1) % n;                                          // 循环邻居
                if ((nums[i] + 1) % n != nums[nxt]) up++;                       // 升序违规
                if ((nums[i] - 1 + n) % n != nums[nxt]) down++;                 // 降序违规
            }
            return up <= 1 || down <= 1;                                        // 等价 up == 0 || down == 0
        }

        int minOperations(vector<int>& nums) {
            if (!test(nums)) return -1;                                         // 不可达
            int n = nums.size();
            int maxIdx = 0;
            bool sorted = true;
            for (int i = 1; i < n; i++) {
                if (nums[i] < nums[i - 1]) sorted = false;
                if (nums[i] > nums[maxIdx]) maxIdx = i;
            }
            if (sorted) return 0;                                               // 已排序
            return min(maxIdx + 1, n - maxIdx + 1);                             // 两策略取 min
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minOperations(self, nums: list[int]) -> int:
            n = len(nums)

            def feasible() -> bool:
                if n == 1:
                    return True
                # 用生成器 + sum 一行算"违规对数". 等价 C++ 双 for 累加
                up   = sum(1 for i in range(n) if (nums[i] + 1) % n != nums[(i + 1) % n])
                down = sum(1 for i in range(n) if (nums[i] - 1) % n != nums[(i + 1) % n])
                return up == 0 or down == 0

            if not feasible():
                return -1
            # max 自带返回值, 再用 index 拿位置; 等价 C++ 一遍扫
            max_idx = nums.index(max(nums))
            if nums == sorted(nums):
                return 0
            return min(max_idx + 1, n - max_idx + 1)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var minOperations = function(nums) {
        const n = nums.length;
        if (n === 1) return 0;

        let up = 0, down = 0;
        for (let i = 0; i < n; i++) {
            const nxt = (i + 1) % n;
            if ((nums[i] + 1) % n !== nums[nxt]) up++;
            if ((nums[i] - 1 + n) % n !== nums[nxt]) down++;
        }
        if (up > 0 && down > 0) return -1;                                      // 不可达

        let maxIdx = 0, sorted = true;
        for (let i = 1; i < n; i++) {
            if (nums[i] < nums[i - 1]) sorted = false;
            if (nums[i] > nums[maxIdx]) maxIdx = i;
        }
        if (sorted) return 0;
        return Math.min(maxIdx + 1, n - maxIdx + 1);
    };
    ```

## Complexity

- **Time**: O(n) — 两遍线性扫.
- **Space**: O(1).

## 相关题目

- 0048\. Rotate Image (待补) — 数组旋转 (二维), 同款"原地操作 + 反转"
- 0189\. Rotate Array (待补) — 数组循环右移, 反转三段技巧
- 0033\. Search in Rotated Sorted Array (待补) — 循环移位数组的二分搜索
- 0153\. Find Minimum in Rotated Sorted Array (待补) — 循环移位数组找最小, 二分
- 0541\. Reverse String II (待补) — 字符串分段反转
- 0042\. Trapping Rain Water (待补) — 同款"双策略对称取 min" 思想

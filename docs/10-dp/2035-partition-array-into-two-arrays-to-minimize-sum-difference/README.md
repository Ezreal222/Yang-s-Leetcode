# 2035. Partition Array Into Two Arrays to Minimize Sum Difference / 将数组分成两个数组并最小化和的差

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Meet in the Middle, Bitmask, Sort, Binary Search · 折半搜索, 状压, 排序, 二分
    - **Link**: [LeetCode](https://leetcode.com/problems/partition-array-into-two-arrays-to-minimize-sum-difference/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 数组 `nums` 长度 `2n` (n ≤ 15). 把它分成**两组各 n 个元素**, 求两组**和之差的绝对值** 的最小值.

**中文**: 长度 `2n` 的 `nums`, 分两组各 `n` 个, 求两组和的差的绝对值最小.

## Key Insights

1. **为什么暴力 / 普通 DP 都不行 / Why brute & knapsack both fail**

    - **暴力枚举**: 选 n 个出来 = `C(2n, n)`. n=15 时 ≈ 1.5×10^8, 边界但容易超时, 且每次 O(n) 求和总共 ≈ 4×10^9.
    - **整体 bitmask**: `2^(2n)` = `2^30` ≈ 10^9, **直接爆**.
    - **背包 DP**: 元素值可达 10^7, 和的范围 3×10^8, dp 表开不下. → **不能用背包**.

    > 看到 "n ≤ 15" 且背包不行 → 立刻想 [折半搜索](../topic-meet-in-the-middle.md).

2. **目标函数变形: 最小 |差| ⟺ 让一组和最接近 total/2 / Min diff ⟺ closest to total/2**

    令选出来那一组和为 `S`, 另一组就是 `total - S`. 差的绝对值:

    $$\text{diff} = |S - (\text{total} - S)| = |\text{total} - 2S|$$

    要 diff 最小 ⟺ `S` 尽量接近 `total/2`. **优化目标只跟"一组和" 有关**.

3. **折半: 左右各 n 个, 一个 mask 同时切两半 / Same mask carves both halves**

    把 `nums` 砍成左右两半各 `n` 个 (索引 `[0, n)` 和 `[n, 2n)`). 对 `mask` 从 `0` 到 `2^n - 1`:

    - `leftSums[cnt]` 收集"左半按 mask 选" 的和 (cnt = mask 里 1 的个数).
    - `rightSums[cnt]` 收集"右半按同 mask 选" 的和.

    > 妙处: 同一个 mask 同时枚举左右. 当 mask 跑遍 `0..2^n-1`, leftSums[k] 自动收齐"左半所有大小为 k 的子集和", rightSums[k] 同理. 互相独立 (因为枚举遍历了全部组合).

4. **配对约束: 左选 k + 右选 n-k = 共 n / Bucket by popcount, pair sizes summing to n**

    "**一组必须 n 个**" 的约束在这里被拆成: 左选 `k`, 右选 `n-k`, k 从 0 到 n. 所以排序 `rightSums[n-k]`, 再对每个 `sumL ∈ leftSums[k]` 二分查最接近 `total/2 - sumL` 的 `sumR`.

    > **按 popcount 分桶** 是带"个数约束" 折半的标配, 没有它配对会乱.

5. **二分定位 + 看左右两个候选 / Check both sides of lower_bound**

    `lower_bound` 返回的位置可能是"刚好等于 target 或第一个大于 target" 的索引. 最近的可能是这个, 也可能是它前一个 (小于 target 那侧). 所以 `d ∈ {-1, 0}` 各试一次, 取小的.

    > 二分找"最接近" 的标准操作 — 找到分界点后两侧都要看.

6. **位运算细节: `mask & (1 << i)` 不是 `mask && (1 << i)` / `&` not `&&`**

    `&` 是按位与, 取出第 i 位; `&&` 是逻辑与, 把两个值当 bool. `mask=4, i=0`: `4 & 1 = 0` (位 0 为 0); `4 && 1 = 1` (4 是真, 1 是真). 用错就**永远拿到 1 或 0**, 整个 mask 解析失效.

    > 状压 / bitmask 题最高频翻车. Python 没有 `&&`, 不会犯; C++/JS 都要小心.

7. **可选简化: `cnt = __builtin_popcount(mask)` / Replace manual count**

    Yang 用 `cnt++` 边扫边数, 完全 OK. 一行替代:

    ```cpp
    int cnt = __builtin_popcount(mask);   // GCC 内置, O(1) 数 mask 里的 1
    ```

    Clang/MSVC 也支持. Python: `mask.bit_count()` (3.10+) 或 `bin(mask).count('1')`.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minimumDifference(vector<int>& nums) {
            int n = nums.size() / 2;
            int total = accumulate(nums.begin(), nums.end(), 0);

            // 折半: 对左/右两半各 2^n 种子集, 按"选了几个" 分桶
            vector<vector<int>> leftSums(n + 1);
            vector<vector<int>> rightSums(n + 1);

            for (int mask = 0; mask < (1 << n); mask++) {
                int sumL = 0, sumR = 0, cnt = 0;
                for (int i = 0; i < n; i++) {
                    if (mask & (1 << i)) {                         // & 不是 &&!
                        cnt++;
                        sumL += nums[i];
                        sumR += nums[i + n];                       // 同 mask 同步切右半
                    }
                }
                leftSums[cnt].push_back(sumL);
                rightSums[cnt].push_back(sumR);
            }

            // 二分准备: 只排右桶
            for (int k = 0; k <= n; k++) {
                sort(rightSums[k].begin(), rightSums[k].end());
            }

            int result = INT_MAX;
            // 配对: 左选 k 个 + 右选 n-k 个 = 共 n 个
            for (int k = 0; k <= n; k++) {
                for (int sumL : leftSums[k]) {
                    int targetR = total / 2 - sumL;                // 让 sumL + sumR ≈ total/2
                    auto& candidates = rightSums[n - k];
                    int idx = lower_bound(candidates.begin(), candidates.end(), targetR)
                              - candidates.begin();
                    // 检查左右各一个候选 (最接近的在 idx-1 或 idx)
                    for (int d = -1; d <= 0; d++) {
                        int i = idx + d;
                        if (i >= 0 && i < (int)candidates.size()) {
                            int groupSum = sumL + candidates[i];
                            result = min(result, abs(total - 2 * groupSum));
                        }
                    }
                }
            }
            return result;
        }
    };
    ```

=== "Python"
    ```python
    from bisect import bisect_left

    class Solution:
        def minimumDifference(self, nums: list[int]) -> int:
            n = len(nums) // 2
            total = sum(nums)
            # defaultdict(list) 也行, 这里用预先 n+1 个桶更直白
            left_sums = [[] for _ in range(n + 1)]
            right_sums = [[] for _ in range(n + 1)]

            for mask in range(1 << n):
                # bit_count() Python 3.10+; 等价 C++ __builtin_popcount
                cnt = mask.bit_count()
                sl = sr = 0
                for i in range(n):
                    if mask & (1 << i):
                        sl += nums[i]
                        sr += nums[i + n]
                left_sums[cnt].append(sl)
                right_sums[cnt].append(sr)

            # 只排右桶, 左桶用来遍历
            for k in range(n + 1):
                right_sums[k].sort()

            result = float('inf')
            for k in range(n + 1):
                cands = right_sums[n - k]
                for sl in left_sums[k]:
                    target_r = total // 2 - sl
                    # bisect_left = C++ lower_bound, 返回第一个 ≥ target 的位置
                    idx = bisect_left(cands, target_r)
                    for d in (-1, 0):
                        i = idx + d
                        if 0 <= i < len(cands):
                            group_sum = sl + cands[i]
                            result = min(result, abs(total - 2 * group_sum))
            return result
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var minimumDifference = function(nums) {
        const n = nums.length / 2;
        const total = nums.reduce((s, x) => s + x, 0);

        // Array.from({length: n+1}, () => []) 创建 n+1 个独立空数组
        // 别写 Array(n+1).fill([]) — fill 会让所有桶指向同一个数组!
        const leftSums = Array.from({length: n + 1}, () => []);
        const rightSums = Array.from({length: n + 1}, () => []);

        for (let mask = 0; mask < (1 << n); mask++) {
            let sl = 0, sr = 0, cnt = 0;
            for (let i = 0; i < n; i++) {
                if (mask & (1 << i)) {
                    cnt++;
                    sl += nums[i];
                    sr += nums[i + n];
                }
            }
            leftSums[cnt].push(sl);
            rightSums[cnt].push(sr);
        }

        // 升序排. compareFn 不能漏, 默认是字典序!
        for (let k = 0; k <= n; k++) rightSums[k].sort((a, b) => a - b);

        // 手写 lower_bound (JS 没有原生二分)
        const lowerBound = (arr, target) => {
            let lo = 0, hi = arr.length;
            while (lo < hi) {
                const mid = (lo + hi) >> 1;
                if (arr[mid] < target) lo = mid + 1;
                else hi = mid;
            }
            return lo;
        };

        let result = Infinity;
        for (let k = 0; k <= n; k++) {
            const cands = rightSums[n - k];
            for (const sl of leftSums[k]) {
                const targetR = Math.floor(total / 2) - sl;
                const idx = lowerBound(cands, targetR);
                for (const d of [-1, 0]) {
                    const i = idx + d;
                    if (i >= 0 && i < cands.length) {
                        const groupSum = sl + cands[i];
                        result = Math.min(result, Math.abs(total - 2 * groupSum));
                    }
                }
            }
        }
        return result;
    };
    ```

## Complexity

- **Time**: O(2^n × n + 2^n × log(2^n)) = O(2^n × n) — 枚举 + 二分.
- **Space**: O(2^n) — 桶存所有子集和.

> n=15: `2^15 × 15 ≈ 5×10^5`. 跟原暴力 `2^30 ≈ 10^9` 差三个数量级.

## 相关题目

- [§10 DP · 折半搜索 (Meet in the Middle) 技巧总结](../topic-meet-in-the-middle.md) — 母方法
- 1755\. Closest Subsequence Sum (待补) — 折半纯模板, 不带"个数约束", 比本题简单
- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 同 "分两堆最小差", 但用 0/1 背包路线 (值小 n 大)
- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 同模型, 判定相等
- 956\. Tallest Billboard (待补) — 同 "分两堆" 思想, 三状态 DP 或折半

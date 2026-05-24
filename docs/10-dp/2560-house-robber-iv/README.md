# 2560. House Robber IV / 打家劫舍 IV

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search on Answer, Greedy, Array · 二分答案, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/house-robber-iv/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 一排房子 `nums`, 偷相邻两间会触发警报. 必须**至少偷 k 间**. 定义"能力 (capability)" 为偷的所有房子里的**最大值**. 求最小可能的 capability.

**中文**: 至少偷 `k` 间不相邻的房子, capability 是偷的最大值, 求最小化的 capability.

## Key Insights

1. **⚠ 这题不是 DP, 是"二分答案 + 贪心" / Despite the name, this is BSA + greedy, not DP**

    系列前三 ([0198](../0198-house-robber/README.md) / [0213](../0213-house-robber-ii/README.md) / [0337](../0337-house-robber-iii/README.md)) 都是 DP 求"最大金额", 本题问的是 **"最小化最大值"**, 解法完全换轨道. 放在 §10 DP 只是为了跟系列同框.

    > LC 命名容易误导. **看"求什么"** (最大金额 vs 最小化最大值) 比题目名更准.

2. **🔑 "最小化最大值" → 二分答案 (BSA) / "Minimize the maximum" → binary search on answer**

    这是一个**强信号**模板:

    > **求"满足某条件的最小 X"** → 二分 X 的可行范围, 判定函数 `canAchieve(X)` 是否成立.

    本题的 X 是 capability. 把 `[min(nums), max(nums)]` 当搜索空间, 二分找**最小的 X** 使得"用能力 X 能偷 ≥ k 间不相邻的 ≤ X 的房子".

    判定函数**单调**: capability 越大越容易满足 → 适合二分.

    > 同模板的题: 0410 分割数组的最大值, 0875 爱吃香蕉的珂珂, 1011 在 D 天内送达包裹的能力, 1482 最小天数等. 看到"最小化最大 / 最大化最小" 就立刻反应 BSA.

3. **判定函数: 贪心 + 跳过相邻 / canRob = greedy, take-if-≤-cap-then-skip-1**

    给定 cap, 怎么算"能偷几间"? 贪心扫:

    - 当前房子 `nums[i] ≤ cap` → **偷**, 跳过 `i+1`, 看 `i+2`.
    - 否则 → **跳**, 看 `i+1`.

    返回偷了几间, 跟 `k` 比.

    **贪心正确性 (交换论证)**: 若某最优解在某位置 i 没偷 (但 `nums[i] ≤ cap`), 换成偷 i 后, "下一可偷位置" 从 ≥ i+1 变成 ≥ i+2, 后续可选范围只是少了 i+1 一个 — 至多损失一间, 但本步多偷一间 → **改成偷不会更差**. 所以贪心成立.

4. **二分模板: "找第一个 true" 写法 / Lower-bound on predicate**

    `canRob(cap)` 关于 `cap` 单调: false...false [true...true]. 找**第一个 true**:

    ```cpp
    while (left < right) {
        int mid = left + (right - left) / 2;       // 防溢出
        if (canRob(mid, k)) right = mid;           // mid 满足 → 解在 [left, mid]
        else                left = mid + 1;        // mid 不满足 → 解在 [mid+1, right]
    }
    return left;                                   // left == right, 即第一个 true
    ```

    > 二分写法的关键: `mid` 落在哪一侧, 决定怎么收 left/right. "找第一个 true" 用 `right = mid` (不缩 mid 本身), 防止跳过解.

5. **`i++` 嵌在 for 里: 一种省事写法 / Two `i++` in one iteration**

    Yang 的 `canRob` 内层:

    ```cpp
    for (int i = 0; i < nums.size(); i++) {
        if (nums[i] <= cap) {
            count++;
            i++;                                   // 再跳一格 → 配合 for 的 i++, 共跳 2 格
        }
    }
    ```

    `if` 里的 `i++` + for 的 `i++` = 偷了一间后跳 2 格 (i → i+2), 跳过相邻. 简洁但容易让读者反应不过来. 等价写法:

    ```cpp
    int i = 0;
    while (i < n) {
        if (nums[i] <= cap) { count++; i += 2; }
        else                                       i += 1;
    }
    ```

    while 版更显式, 更"阅读友好".

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minCapability(vector<int>& nums, int k) {
            int left  = *min_element(nums.begin(), nums.end());
            int right = *max_element(nums.begin(), nums.end());
            while (left < right) {
                int mid = left + (right - left) / 2;
                if (canRob(nums, mid, k)) right = mid;             // 找第一个 true
                else                       left = mid + 1;
            }
            return left;
        }
    private:
        bool canRob(vector<int>& nums, int cap, int k) {
            int count = 0, n = nums.size();
            for (int i = 0; i < n; i++) {
                if (nums[i] <= cap) {
                    count++;
                    i++;                                           // 跳过相邻
                    if (count >= k) return true;                   // 早返
                }
            }
            return count >= k;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minCapability(self, nums: list[int], k: int) -> int:
            # 二分答案的边界: 偷的最大值至少是 min(nums) (至少一间), 至多是 max(nums)
            lo, hi = min(nums), max(nums)

            def can_rob(cap: int) -> bool:
                # 贪心扫: 遇到 ≤cap 就偷, 跳过下一间
                count, i, n = 0, 0, len(nums)
                while i < n:
                    if nums[i] <= cap:
                        count += 1
                        if count >= k:
                            return True
                        i += 2                                     # 偷了, 跳过相邻
                    else:
                        i += 1
                return count >= k

            # 二分找第一个 can_rob 为 True 的 cap
            while lo < hi:
                mid = (lo + hi) // 2
                if can_rob(mid):
                    hi = mid
                else:
                    lo = mid + 1
            return lo
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {boolean}
     */
    var minCapability = function(nums, k) {
        // Math.min/max 接散参; 用 ...spread 把数组展开, 数组超大时可能爆栈, 但本题范围 OK
        let lo = Math.min(...nums), hi = Math.max(...nums);

        const canRob = (cap) => {
            let count = 0, i = 0, n = nums.length;
            while (i < n) {
                if (nums[i] <= cap) {
                    count++;
                    if (count >= k) return true;
                    i += 2;
                } else {
                    i++;
                }
            }
            return count >= k;
        };

        while (lo < hi) {
            const mid = (lo + hi) >> 1;                            // 位运算除 2, 对正数等价 Math.floor
            if (canRob(mid)) hi = mid;
            else             lo = mid + 1;
        }
        return lo;
    };
    ```

## Complexity

- **Time**: O(n × log(max - min)) — 外层二分 log(值域), 内层 O(n) 贪心扫.
- **Space**: O(1).

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) / [0213](../0213-house-robber-ii/README.md) / [0337](../0337-house-robber-iii/README.md) — 系列前三, DP 求最大金额
- 0410\. Split Array Largest Sum (待补) — 同模板"最小化最大值" BSA
- 0875\. Koko Eating Bananas (待补) — 同模板, 二分速度 + 贪心检查
- 1011\. Capacity To Ship Packages Within D Days (待补) — 同模板, 二分容量
- 1482\. Minimum Number of Days to Make m Bouquets (待补) — 同模板, 二分天数

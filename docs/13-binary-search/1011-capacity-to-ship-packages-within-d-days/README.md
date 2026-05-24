# 1011. Capacity To Ship Packages Within D Days / 在 D 天内送达包裹的能力

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search on Answer, Greedy, Array · 二分答案, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 传送带上 `weights` 个包裹**按顺序**装船, 求**最小船载重量** 使得能在 `days` 天内全部送达. 每天装的包裹必须**连续**, 总重 ≤ 船载.

**中文**: 按序装船, 求最小船载使得 `days` 天内能全部运完 (每天连续装, 不超船载).

## Key Insights

1. **🔑 "最小化最大值" 模式 → 二分答案 (BSA) / "Minimize the maximum" template**

    跟 [2560 House Robber IV](../2560-house-robber-iv/README.md) 同款 BSA + 贪心 check 模板:

    > **求"满足某条件的最小 X"** + 条件关于 X 单调 → 二分 X 的可行范围, `canAchieve(X)` 做判定.

    本题 X 是船载量. capacity 越大越容易在 days 天内运完 → 单调, 适合二分.

2. **搜索区间: `[max(weights), sum(weights)]` / Search bounds**

    - **下界** `max(weights)`: 至少要能放下最重那个包裹 (否则永远运不走它).
    - **上界** `sum(weights)`: 一天能装完所有 → 1 天必能完成, 一定可行.

    > 这两个边界是 BSA 的标配 — **下界是"必要条件", 上界是"必充分条件"**. 任何 BSA 都先想这两个.

3. **判定函数: 贪心模拟 / canShip = greedy simulation**

    给定 cap, 一遍扫:

    - 当前累计 `curLoad + w > cap` → 这天装满, 新开一天, `curLoad = w`.
    - 否则 → 继续装, `curLoad += w`.

    需要的天数 `needDays ≤ days` 则可行.

    **贪心正确性**: 每天能多塞就多塞 (不超 cap). 若提前换天, 后面包裹只可能更晚装完 → **永远不会更好**.

4. **二分模板: "找第一个 true" / Lower-bound on predicate**

    `canShip(cap)` 关于 cap 单调: `false...false [true...true]`. 找第一个 true:

    ```cpp
    while (left < right) {
        int mid = left + (right - left) / 2;       // 防溢出
        if (canShip(mid)) right = mid;             // mid 满足 → 解在 [left, mid]
        else              left = mid + 1;          // mid 不满足 → 解在 [mid+1, right]
    }
    return left;
    ```

    > 跟 2560 同代码骨架, 换个判定就过. **写一次, 后面所有 BSA 题都套这套**.

5. **跟"区间和的二分" (0410) 完全等价 / Equivalent to LC 410**

    0410 "Split Array Largest Sum" 把数组分成 m 段, 求各段和最大值的**最小值**. 把"包裹" 当数组元素, "天" 当段数, 本题 == 0410. 同代码一份能过两道.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int shipWithinDays(vector<int>& weights, int days) {
            int left  = *max_element(weights.begin(), weights.end()); // 最重包裹是下界
            int right = accumulate(weights.begin(), weights.end(), 0); // 全装一天是上界
            while (left < right) {
                int mid = left + (right - left) / 2;
                if (canShip(weights, mid, days)) right = mid;
                else                              left = mid + 1;
            }
            return left;
        }
    private:
        bool canShip(vector<int>& weights, int cap, int days) {
            int needDays = 1, curLoad = 0;
            for (int w : weights) {
                if (curLoad + w > cap) {                              // 装不下了, 新开一天
                    needDays++;
                    curLoad = w;
                } else {
                    curLoad += w;
                }
            }
            return needDays <= days;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def shipWithinDays(self, weights: list[int], days: int) -> int:
            lo, hi = max(weights), sum(weights)

            def can_ship(cap: int) -> bool:
                # 贪心: 能塞就塞, 装不下新开一天
                need_days, cur = 1, 0
                for w in weights:
                    if cur + w > cap:
                        need_days += 1
                        cur = w
                    else:
                        cur += w
                return need_days <= days

            # 二分找第一个 can_ship 为 True 的 cap
            while lo < hi:
                mid = (lo + hi) // 2
                if can_ship(mid):
                    hi = mid
                else:
                    lo = mid + 1
            return lo
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} weights
     * @param {number} days
     * @return {number}
     */
    var shipWithinDays = function(weights, days) {
        // reduce 求总和 — 等价 C++ accumulate
        let lo = Math.max(...weights);
        let hi = weights.reduce((s, w) => s + w, 0);

        const canShip = (cap) => {
            let needDays = 1, cur = 0;
            for (const w of weights) {
                if (cur + w > cap) { needDays++; cur = w; }
                else                                cur += w;
            }
            return needDays <= days;
        };

        while (lo < hi) {
            const mid = (lo + hi) >> 1;                            // 位运算除 2
            if (canShip(mid)) hi = mid;
            else              lo = mid + 1;
        }
        return lo;
    };
    ```

## Complexity

- **Time**: O(n × log(sum - max)) — 外层 BSA, 内层 O(n) 贪心扫.
- **Space**: O(1).

## 相关题目

- [2560. House Robber IV](../2560-house-robber-iv/README.md) — 同 BSA 模板 (二分能力 + 贪心)
- 0410\. Split Array Largest Sum (待补) — 跟本题完全等价, 同代码一份过两题
- 0875\. Koko Eating Bananas (待补) — 同模板, 二分吃速 + 贪心算时间
- 1482\. Minimum Number of Days to Make m Bouquets (待补) — 同模板, 二分天数
- 0668\. Kth Smallest Number in Multiplication Table (待补) — BSA 进阶

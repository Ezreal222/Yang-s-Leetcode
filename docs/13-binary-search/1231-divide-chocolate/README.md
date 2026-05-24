# 1231. Divide Chocolate / 分享巧克力

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Binary Search on Answer, Greedy, Array · 二分答案, 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/divide-chocolate/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 巧克力按顺序分成连续段 (要切 `k` 刀, 共 `k+1` 段, 你和 `k` 位朋友各拿一段). 你只拿"**总甜度最小的那段**". 求"**你拿的那段** 的甜度最大值".

**中文**: 把 `sweetness` 切 `k` 刀分 `k+1` 段, 你拿最小那段. 求你拿那段甜度的**最大可能值**.

## Key Insights

1. **🔑 "最大化最小值" → BSA 的镜像版 / "Maximize the minimum" — mirror of "minimize the maximum"**

    跟 [2560 House Robber IV](../2560-house-robber-iv/README.md) / [1011 船载](../1011-capacity-to-ship-packages-within-d-days/README.md) 的"**最小化最大值**" 互为镜像:

    | 模式 | 例题 | 二分目标 | check |
    |---|---|---|---|
    | **最小化最大值** | 2560 / 1011 | "X 够大吗?" 单调 false→true | 找**第一个 true** |
    | **最大化最小值** (本题) | 1231 | "X 够小吗?" 单调 true→false | 找**最后一个 true** |

    两个方向**判定函数都单调**, 但找的端点相反 → **二分模板也不同**.

2. **状态: `canDivide(limit)` = 能否切出至少 k+1 段, 每段总甜 ≥ limit / Predicate**

    给定 limit, 贪心扫:

    - 当前累计 `cur + s ≥ limit` → 切一刀, `cur = 0`, `pieces++`.
    - 否则继续累加.

    返回 `pieces ≥ k + 1` (k 个朋友 + 自己 = k+1 段).

    **贪心正确性**: 一旦 `cur ≥ limit`, **立刻切** vs 多塞几块再切, 立刻切让后面剩更多原料 → 多切几段 → 只会让 `pieces ≥ k+1` 更容易满足. **早切不劣**.

3. **🔑 "找最后一个 true" 二分模板 / Upper-bound binary search**

    `canDivide(limit)` 关于 limit 单调: `true...true [false...false]`. 找**最后一个 true** (最大可行 limit). 模板跟"找第一个 true" 不一样, 容易翻车:

    ```cpp
    while (left < right) {
        int mid = left + (right - left + 1) / 2;       // ⚠ +1 (向上取整)
        if (canDivide(mid)) left = mid;                // mid 满足 → 解在 [mid, right]
        else                right = mid - 1;           // mid 不满足 → 解在 [left, mid-1]
    }
    return left;
    ```

    **两个跟"第一个 true" 模板的差异**:

    | 差异点 | 找第一个 true (2560/1011) | 找最后一个 true (本题) |
    |---|---|---|
    | mid 计算 | `left + (right-left) / 2` (向下取整) | `left + (right-left+1) / 2` (**向上取整**) |
    | true 时 | `right = mid` | `left = mid` |
    | false 时 | `left = mid + 1` | `right = mid - 1` |

    **为什么 mid 要向上取整?** 若用向下取整, 当 `left + 1 == right` 且 `canDivide(mid)` 为 true 时, `mid = left`, 然后 `left = mid = left` → **死循环**. 向上取整让 mid 偏向 right, 避免这个 corner case.

    > **背口诀**: "**第一个 true → 向下取整 + right=mid; 最后一个 true → 向上取整 + left=mid**". 写错就死循环或漏解.

4. **搜索区间: `[min(sweetness), sum(sweetness)]` / Search bounds**

    - **下界** `min(sweetness)`: 你拿的那段至少包含一块, 必 ≥ 最小那块.
    - **上界** `sum(sweetness)`: 不切刀 (k=0) 时你独享整条 — 但题目至少有一个朋友, 实际上下界保守用 sum 即可.

    > 同 [1011](../1011-capacity-to-ship-packages-within-d-days/README.md): **下界 = 必要条件, 上界 = 必充分条件**.

5. **k+1 段, 不是 k 段 / k+1 pieces, not k**

    "切 k 刀 = 分 k+1 段". 漏掉 +1 直接 WA. Yang 已经 +1 处理.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maximizeSweetness(vector<int>& sweetness, int k) {
            int left  = *min_element(sweetness.begin(), sweetness.end());
            int right = accumulate(sweetness.begin(), sweetness.end(), 0);
            while (left < right) {
                int mid = left + (right - left + 1) / 2;           // ⚠ 向上取整, 找最后一个 true 必备
                if (canDivide(sweetness, mid, k)) left = mid;
                else                              right = mid - 1;
            }
            return left;
        }
    private:
        bool canDivide(vector<int>& sweetness, int limit, int k) {
            int pieces = 0, cur = 0;
            for (int s : sweetness) {
                if (cur + s >= limit) {                            // 满了立刻切, 早切不劣
                    pieces++;
                    cur = 0;
                } else {
                    cur += s;
                }
            }
            return pieces >= k + 1;                                // k 刀 = k+1 段
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maximizeSweetness(self, sweetness: list[int], k: int) -> int:
            lo, hi = min(sweetness), sum(sweetness)

            def can_divide(limit: int) -> bool:
                pieces, cur = 0, 0
                for s in sweetness:
                    if cur + s >= limit:
                        pieces += 1
                        cur = 0
                    else:
                        cur += s
                return pieces >= k + 1                             # k 刀 = k+1 段

            # 找最后一个 true: 向上取整 mid + true 时 lo=mid
            while lo < hi:
                mid = (lo + hi + 1) // 2                           # ⚠ +1
                if can_divide(mid):
                    lo = mid
                else:
                    hi = mid - 1
            return lo
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} sweetness
     * @param {number} k
     * @return {number}
     */
    var maximizeSweetness = function(sweetness, k) {
        let lo = Math.min(...sweetness);
        let hi = sweetness.reduce((s, x) => s + x, 0);

        const canDivide = (limit) => {
            let pieces = 0, cur = 0;
            for (const s of sweetness) {
                if (cur + s >= limit) { pieces++; cur = 0; }
                else                                cur += s;
            }
            return pieces >= k + 1;
        };

        while (lo < hi) {
            const mid = (lo + hi + 1) >> 1;                        // ⚠ +1 向上取整
            if (canDivide(mid)) lo = mid;
            else                hi = mid - 1;
        }
        return lo;
    };
    ```

## Complexity

- **Time**: O(n × log(sum - min)).
- **Space**: O(1).

## 相关题目

- [2560. House Robber IV](../2560-house-robber-iv/README.md) — "最小化最大值" 镜像模板
- [1011. Capacity To Ship Packages Within D Days](../1011-capacity-to-ship-packages-within-d-days/README.md) — "最小化最大值" 镜像模板
- 0410\. Split Array Largest Sum (待补) — 同 1011 的"最小化最大值"
- 0875\. Koko Eating Bananas (待补) — "最小化最大值" 同模板
- 1482\. Minimum Number of Days to Make m Bouquets (待补) — "最小化最大值" 同模板
- 0668\. Kth Smallest Number in Multiplication Table (待补) — BSA 进阶

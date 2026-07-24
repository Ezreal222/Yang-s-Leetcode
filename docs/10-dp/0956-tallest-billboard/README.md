# 0956. Tallest Billboard / 最高的广告牌

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, State Compression, Hash Map, Knapsack-like · 动态规划, 状态压缩, 哈希表, 类背包
    - **Link**: [LeetCode](https://leetcode.com/problems/tallest-billboard/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Max height of two equal-tall poles built from subset of rods (rest unused)** → **DP with `diff` as state**: `dp[diff] = max shorter side reachable`. Each rod: **skip** / **add to taller side** (`diff + x`) / **add to shorter side** (`|diff - x|`, shorter += min(diff, x)). Answer = `dp[0]`.
>
> **中文**: **两根等高杆的最高高度 (从 rods 子集拼)** → **DP 用 diff 当状态**: `dp[diff]` = 该差值下"较短侧" 最大高度. 每根 x 三选: **不放** / **放长侧** (`diff + x`) / **放短侧** (`|diff - x|`, shorter += min(diff, x)). 答案 = `dp[0]`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给一组杆 `rods` (可**丢弃**部分). 用它们拼**两根**杆, **必须等高**, 求**最高高度** (无解返 0).

**中文**: 从 rods 里选一部分, 拼出两根等高杆, 求最高的等高高度.

## Key Insights

1. **🔑 朴素思路 3^n → 状态 DP / Brute 3^n → state DP**

    每根杆 3 种选择: 放左 / 放右 / 不放. 暴力 **3^n** (n ≤ 20 → 3.5 × 10^9, TLE).

    → 需要**状态压缩**: 不关心具体每侧多高, 只关心**差值 diff = |left - right|**.

    > **"只关心相对量" → 状态从 O(H²) 压到 O(H)** — 关键洞察.

2. **🔑 灵魂定义: `dp[diff] = 该差值下较短侧的最大高度` / dp[diff] = max shorter side**

    - 若 dp[diff] = s, 则**存在一种拼法** 让**较短侧 = s, 较长侧 = s + diff**.
    - **最终答案**: `dp[0]` — 差值 0 时较短 = 较长 = 两侧共同高度.

    > **为啥追踪 shorter 而非 longer?** — 因为 **longer = shorter + diff** 直接推出. 只需一个变量.

3. **🔑 三种转移 / Three transitions per rod**

    对每根 x, 从 (diff, shorter) 出发:

    **① 不放**: `(diff, shorter)` 不变.

    **② 放到较长侧**: 长侧 += x, diff 增 x → **`(diff + x, shorter)`**.

    **③ 放到较短侧**: 短侧 += x. 两种子情况:

    - **`x < diff`** (短侧仍是短): `(diff - x, shorter + x)`.
    - **`x >= diff`** (短侧超越了长侧, 身份互换): 新长 = 短 + x, 新短 = 原长 = shorter + diff. `(x - diff, shorter + diff)`.

    **合并** (关键漂亮): 无论哪种子情况:

    - **新 diff = `|diff - x|`**.
    - **新 shorter = `shorter + min(diff, x)`** — "新短侧" 涨了 min 值.

    > **"分情况后合并成绝对值 + min"** 是这题的第二个灵魂洞察.

4. **🔑 hash map 而非 array — 因为 diff 分布稀疏 / Use hash map, not array**

    diff 可能到 sum(rods) ≤ 5000. 数组也可以, 但**大部分 diff 不可达** → hash 更省.

    ```cpp
    unordered_map<int, int> dp;
    dp[0] = 0;                                          // 初始: 两侧都空, diff = 0, shorter = 0
    ```

    每处理一根杆, **新 dp** 从**旧 dp** 拷贝 (代表"不放" 分支), 再套用 ② ③ 转移:

    ```cpp
    unordered_map<int, int> next = dp;                  // 分支 ① "不放"
    for (auto& [diff, shorter] : dp) {
        // 分支 ②: 放长侧
        next[diff + x] = max(next[diff + x], shorter);
        // 分支 ③: 放短侧 (合并公式)
        int newDiff = abs(diff - x);
        int newShorter = shorter + min(diff, x);
        next[newDiff] = max(next[newDiff], newShorter);
    }
    dp = move(next);
    ```

    - **`max(...)` 合并**: 同一 diff 可能有多种到达方式, 取 shorter 最大的.
    - **`move(next)`** — C++ 右值移动, 免拷贝, 常数优化.

5. **🔑 为啥 `next = dp` 拷贝? 不能原地更新? / Why copy, not in-place**

    - 原地更新会**用到刚才这一轮 x 生成的状态**, 相当于**"这根杆用两次"** — 错.
    - **每根杆只能用一次**, 所以必须**先冻结旧 dp 再算新**.

    > 跟**0/1 背包**的"必须**倒序** j" 是同一个原因 (防止同一物品被使用多次). 这里用**双 map 分离** 达到同样效果.

6. **🔑 答案 `dp[0]` / Return dp[0]**

    最终 diff = 0 时, `dp[0]` = 两侧共同高度 = 答案. 初始 `dp[0] = 0` 保证**"两侧都不拼"** 的 fallback (0 是合法答案).

7. **🔑 复杂度分析 / Complexity**

    - **状态数**: diff ∈ [0, sum(rods)/2]. 每个 diff 只存最大 shorter.
    - **Time**: O(n × sum) — n 根杆, 每次遍历 dp (最多 sum 项).
    - **Space**: O(sum) — hash map.

    > LC constraints: n ≤ 20, sum ≤ 5000 → 20 × 5000 = 10^5, 稳过.

8. **🔑 跟 0/1 背包的关系: 3-way "扩展背包" / vs 0/1 knapsack: 3-way extended**

    经典 0/1 背包: 每物 2 选 (取 / 不取). **本题 3 选** (左 / 右 / 不放). 差别在**状态维度**:

    - 0/1: 单一"容量" 维度.
    - **本题**: **差值** 维度 (相对量).

    > **"扩展背包"** — 每物多种放置方式. 类似的还有[0518 Coin Change II](../0518-coin-change-ii/README.md) 完全背包等家族.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int tallestBillboard(vector<int>& rods) {
            unordered_map<int, int> dp;
            dp[0] = 0;                                                   // 两侧都空
            for (int x : rods) {
                unordered_map<int, int> next = dp;                       // ① "不放" 分支
                for (auto& [diff, shorter] : dp) {
                    // ② 放长侧
                    next[diff + x] = max(next[diff + x], shorter);
                    // ③ 放短侧 (合并公式)
                    int newDiff = abs(diff - x);
                    int newShorter = shorter + min(diff, x);
                    next[newDiff] = max(next[newDiff], newShorter);
                }
                dp = move(next);                                         // 右值移动, 免拷贝
            }
            return dp[0];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def tallestBillboard(self, rods: list[int]) -> int:
            dp = {0: 0}
            for x in rods:
                # copy 冻结旧状态, next 起点包含"不放" 分支
                nxt = dict(dp)
                for diff, shorter in dp.items():
                    # 放长侧
                    nxt[diff + x] = max(nxt.get(diff + x, 0), shorter)
                    # 放短侧
                    new_diff = abs(diff - x)
                    new_shorter = shorter + min(diff, x)
                    nxt[new_diff] = max(nxt.get(new_diff, 0), new_shorter)
                dp = nxt
            return dp[0]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} rods
     * @return {number}
     */
    var tallestBillboard = function(rods) {
        // Map 支持数字 key + 保留插入顺序. 也可用普通 Object
        let dp = new Map([[0, 0]]);
        for (const x of rods) {
            const next = new Map(dp);       // 拷贝作 "不放" 起点
            for (const [diff, shorter] of dp) {
                // 放长侧
                const k1 = diff + x;
                next.set(k1, Math.max(next.get(k1) || 0, shorter));
                // 放短侧
                const newDiff = Math.abs(diff - x);
                const newShorter = shorter + Math.min(diff, x);
                next.set(newDiff, Math.max(next.get(newDiff) || 0, newShorter));
            }
            dp = next;
        }
        return dp.get(0) || 0;
    };
    ```

## Complexity

- **Time**: O(n × S) — n = rods 数, S = sum(rods).
- **Space**: O(S) — hash map.

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP 母题
- [0416. Partition Equal Subset Sum](../0416-partition-equal-subset-sum/README.md) — 0/1 背包判等和
- [1049. Last Stone Weight II](../1049-last-stone-weight-ii/README.md) — 0/1 背包最小差 (跟本题**最接近**! 都是"分两堆")
- [0494. Target Sum](../0494-target-sum/README.md) — 0/1 背包 (+/- 选择)
- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 完全背包组合数
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包最值
- [0887. Super Egg Drop](../0887-super-egg-drop/README.md) — Hard 反向 DP
- [(§10 折半搜索) 2035](../2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) — 折半 + 分两堆最小差
- 0805\. Split Array With Same Average (待补) — 分两组 + DP
- 0879\. Profitable Schemes (待补) — 多维 0/1 背包

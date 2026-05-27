# 0853. Car Fleet / 车队

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Sorting, Greedy · 单调栈, 排序, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/car-fleet/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 道路是一维, 终点在 `target`. `n` 辆车在不同位置 `position[i]`, 各自速度 `speed[i]`, 都向 `target` 行驶. 后车追上前车后**与前车合并** 成同一车队 (按前车速度匀速). 求**最终到达 target 的车队数**.

**中文**: 一维公路, 各车朝终点开. 后车追上前车则合并成一队 (按前车速度). 问到达终点时有几个车队.

## Key Insights

1. **🔑 核心转化: 算每辆车到 target 的时间, 排序按位置 desc 扫 / Time-to-target + sort by pos desc**

    每辆车单独行驶时**到达 target 的时间** = `(target - pos) / speed`.

    把车按**位置从大到小** (离 target 近 → 远) 排序, 然后从近到远扫描:

    - 当前车的 `t` 若 **>** 前面"车队领头" 的 `lastTime` → 它**追不上** → 独立成队, `fleets++`, 更新 `lastTime = t`.
    - 否则 → 它会**追上前队**, 合并 (不计新队, 不更新 lastTime — 因为合并后跟随领头速度, 整队时间仍是领头那个更大的).

    > **领头车 = 队里到达时间最长的那个** (其他车被它"拖慢"). 所以 lastTime 只跟领头有关.

2. **🔑 为什么按位置 desc 扫? / Why descending by position**

    离 target 近的车**永远是潜在领头** (它在前面, 后车要追). 从最近开始, 每个新车要么追得上 (join), 要么追不上 (new fleet) — 决策只看跟最近一次领头的对比.

    若按位置 asc 扫 (从远到近), 关系反过来 — 也能做但更绕. desc 更自然.

3. **🔑 v2 优化: 不需要真"栈", 只要 lastTime + fleets 计数 / Degenerates to single max + counter**

    Yang v1 用 vector 当栈, push 每个新车队的时间. 但**栈里只有栈顶被访问** — 我们只跟"最近一次入栈的时间" 比. 所以**单个变量 lastTime 就够**, 不用 vector.

    这是**"单调栈退化成单变量"** 的典型 — 当你只 care 栈顶时, 栈本身是冗余的.

    > 同款简化: [0674 LCIS](../../10-dp/0674-longest-continuous-increasing-subsequence/README.md) 的 cur 变量 vs 整个 dp 数组. **能压成 O(1) state 就压**.

4. **v2 还把 position + speed 合成 `pair` 排序 / Pair sort instead of separate sort + map**

    Yang v1 用 `unordered_map<position, time>` 存映射, 排序 position 后查 map 拿时间. 多此一举.

    v2 直接 `vector<pair<int, int>> cars(n)` 把 (pos, speed) 配对, **按 pos desc 排序**, 一遍扫即可. 省 map 省一次哈希查.

    > **数据捆绑后排序** 是这类题的简洁写法. 别让 map 替代天然的 pair / struct.

5. **`> ` 严格不等 — 等于时算追上 / Strict greater**

    `t > lastTime` 用严格. 若 `t == lastTime`, 两车**同时到 target**, 算追上 (同队). 改成 `>=` 会把同时到的算两队 — WA.

6. **复杂度 O(n log n) — 排序主导 / Sort dominates**

    扫描线性 O(n), 排序 O(n log n). 整体 O(n log n), O(1) 额外 (v2).

## Solution

=== "C++"
    === "v2 推荐: pair 排序 + O(1) state"
        ```cpp
        class Solution {
        public:
            int carFleet(int target, vector<int>& position, vector<int>& speed) {
                int n = position.size();
                vector<pair<int, int>> cars(n);
                for (int i = 0; i < n; i++) cars[i] = {position[i], speed[i]};
                // pair 默认按 first 比, greater<> 让 first 降序 → 离 target 近的在前
                sort(cars.begin(), cars.end(), greater<>());

                int fleets = 0;
                double lastTime = 0;
                for (auto& [pos, spd] : cars) {
                    double t = (double)(target - pos) / spd;
                    if (t > lastTime) {                            // 追不上, 独立成队
                        fleets++;
                        lastTime = t;                              // 新领头时间
                    }
                    // 否则追上, 合入前队, 不更新 lastTime (整队还是按领头跑)
                }
                return fleets;
            }
        };
        ```

    === "v1 (Yang 原版): map + 单调栈"
        ```cpp
        // 思路对, 但用 map + vector 栈, 比 v2 啰嗦
        class Solution {
        public:
            int carFleet(int target, vector<int>& position, vector<int>& speed) {
                int n = speed.size();
                unordered_map<int, double> m;
                for (int i = 0; i < n; i++) {
                    m[position[i]] = 1.0 * (target - position[i]) / speed[i];
                }
                vector<double> stk;
                sort(position.begin(), position.end(), greater<int>());
                for (int i = 0; i < n; i++) {
                    if (stk.empty() || m[position[i]] > stk.back()) {
                        stk.push_back(m[position[i]]);             // 新车队
                    }
                }
                return stk.size();
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def carFleet(self, target: int, position: list[int], speed: list[int]) -> int:
            # zip + sorted: 一行把两数组配对并按 pos 降序排
            # 等价 C++ vector<pair<int,int>> + sort greater<>
            cars = sorted(zip(position, speed), reverse=True)

            fleets = 0
            last_time = 0.0
            for pos, spd in cars:
                t = (target - pos) / spd
                if t > last_time:
                    fleets += 1
                    last_time = t
            return fleets
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} target
     * @param {number[]} position
     * @param {number[]} speed
     * @return {number}
     */
    var carFleet = function(target, position, speed) {
        const n = position.length;
        const cars = position.map((p, i) => [p, speed[i]]);
        // sort 数字 desc: compareFn (b, a) => a - b
        cars.sort((a, b) => b[0] - a[0]);

        let fleets = 0;
        let lastTime = 0;
        for (const [pos, spd] of cars) {
            const t = (target - pos) / spd;
            if (t > lastTime) {
                fleets++;
                lastTime = t;
            }
        }
        return fleets;
    };
    ```

## Complexity

- **Time**: O(n log n) — 排序.
- **Space**: O(n) — 配对数组 (v2) / O(n) map + vec (v1).

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 单调栈基础
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 模板
- [0901. Online Stock Span](../0901-online-stock-span/README.md) — 流式版单调栈
- 1776\. Car Fleet II (待补) — 进阶版, 求**每辆车被前车追上的时间**
- 0452\. Minimum Number of Arrows to Burst Balloons — 同款"排序 + 一遍扫" 区间贪心 → [§09 0452](../../09-greedy/0452-minimum-number-of-arrows-to-burst-balloons/README.md)
- 0435\. Non-overlapping Intervals — 同款排序后扫描 → [§09 0435](../../09-greedy/0435-non-overlapping-intervals/README.md)

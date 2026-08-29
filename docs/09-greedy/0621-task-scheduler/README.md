# 0621. Task Scheduler / 任务调度器

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Array, Hash Table, Counting, Sorting, Heap · 贪心, 数组, 哈希, 计数, 堆
    - **Link**: [LeetCode](https://leetcode.com/problems/task-scheduler/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Bound the answer with a formula**: dominant task builds a skeleton of `(maxFreq - 1)` "frames" each of length `n + 1`, plus `maxCount` tail tasks. Fillers slide into holes for free. Answer = `max(len(tasks), (maxFreq - 1) * (n + 1) + maxCount)`.
>
> **中文**: **公式秒杀** — 出现次数最多的任务撑起 `(maxFreq - 1)` 个长为 `n + 1` 的"框架" + 末端 `maxCount` 个并列 tie 任务; 其余任务顺势填空. 答案 = `max(总数, (maxFreq - 1) * (n + 1) + maxCount)`.
>
> *Template / 模版*: **Structural bound + tie-count** — 找**瓶颈任务**, 算它撑出的骨架, 再和"直接排完" 取 max.

## Problem

**EN**: 任务用大写字母表示, CPU 每单位时间跑一个 task 或 idle. **同一 task 两次执行必须间隔 ≥ n** 单位. 求最少时间.

**中文**: 冷却间隔 n, 求执行完全部任务的最短时间.

**Example**: `tasks = "AAABBB"`, `n = 2` → `"A B _ A B _ A B"` = 8. (但 `n = 2` 且只有 A/B 时其实可以直接 `A B _ A B _ A B` = 8, 无空槽写法 `A B x A B x A B`.)

## Key Insights

1. **🔑 灵魂: "最忙的任务" 撑起骨架 / Dominant task defines the skeleton**

    设 `maxFreq` = 出现最多次的任务的次数. 那**它自己**就得: 跑 maxFreq 次, 每两次隔 ≥ n. 即:

    ```
    A _ _ ... _ A _ _ ... _ A ... A       ← maxFreq 个 A, 中间各 n 个空
    ```

    → 至少 `(maxFreq - 1) * (n + 1) + 1` 单位. 那些 `_` 就是**填空位** — 让别的任务插进去.

    > **"瓶颈决定下界"** — 贪心 + 数学题常用. 找出 rate-limiting 元素, 剩下的随便安排.

2. **🔑 tie 处理: `maxCount` 个任务并列最多次 / Multiple tasks tied at max**

    若 `A` 和 `B` 都出现 `maxFreq` 次? 每个"框架" 末尾都得**留够 maxCount 个位置**:

    ```
    A B _ ... A B _ ... A B    ← 最后一格从 1 变成 maxCount
    ```

    → 公式: **`(maxFreq - 1) * (n + 1) + maxCount`**.

    - `(maxFreq - 1)` 个完整框架, 每个长 `n + 1`.
    - 末尾追加 `maxCount` 个 tie 任务 (它们只占最后 1 帧).

    > **tie 是最容易漏的点**. 只算 `(maxFreq - 1) * (n + 1) + 1` 遇到 tie 会算少.

3. **🔑 灵魂: 为啥要 `max(公式, len(tasks))` / Why max with total count**

    **反例**: `tasks = "AAABBBCC"`, `n = 0` (无冷却). 公式给 `(3-1)*(0+1) + 2 = 4`, 但实际得 8 (每任务都得跑). → 公式 undercount.

    **本质**: 公式的 `_` 是"预留空槽". 若填空任务数 > 空槽数, 空槽全被填满, 剩余任务只能**排在后面** — 此时总时间 = **任务总数** (无 idle).

    → 结论: **答案 = max(公式, tasks.size())**.

    - 当 n **大** / 主导任务多 → 公式赢, 存在 idle.
    - 当 n **小** / 分布均匀 → 长度赢, 无 idle.

    > **易错点 top 1**: 只用公式不取 max, 遇到 `n = 0` 或任务足够多时 fail.

4. **🔑 为啥公式**始终**是下界 / Why formula is always a lower bound**

    - **maxFreq 任务本身**就得 `(maxFreq - 1) * (n + 1) + 1` 时间 — 无论怎么排.
    - 最后一帧内每有一个 tie 任务, 都得多占 1 单位 → `+ maxCount` 而非 `+ 1`.

    → 公式是**必要下界**. 而任务总数也是下界 (至少每任务跑一次). 取 max 即为答案.

    **为何取 max 也是上界**? 需构造性证明:

    - 若公式 ≥ 总数: 可以按框架填 (idle 不可避免), 完美达到公式.
    - 若总数 ≥ 公式: 空槽全被填满, 无 idle, 时间 = 总数.

    (严格证明可用交换论证 / 反证. 略.)

5. **🔑 O(n) 算法 / Linear scan**

    只需 `freq[26]` 计数 + 一次 `max_element` + 一次 `count`. 全 O(n + 26) = O(n).

    - **不用 heap**! 大部分教程用大根堆模拟 (O(n log 26)), 但**公式解 O(n)** 更短更快.
    - Heap 版本对**扩展题** (如"打印每步实际调度序列") 更有用.

6. **🔑 heap 解法思路 (备用) / Heap simulation as backup**

    每 (n+1) 帧作一轮:

    1. 大根堆按 freq. 每轮 pop 至多 `n + 1` 个, 各 freq -1.
    2. 若 pop 出的**还有剩**, 再 push 回堆.
    3. 若堆空且这轮没填满 `n + 1` → 补 idle 单位.

    - 时间 O(n log 26) ≈ O(n).
    - 优点: 可以顺便**输出实际序列** — 公式解只给数量.
    - 面试若追问"输出调度", 切 heap.

7. **🔑 复杂度 / Complexity**

    - **Time**: O(n) — 一次计数 + 一次找 max + 一次数 tie.
    - **Space**: O(1) — freq 数组固定 26.

8. **🔑 易错点 top 2 / Pitfalls**

    - **忘取 `max(..., tasks.size())`** — n = 0 或高频冷却小时 fail.
    - **忘 tie 计数** — `+1` vs `+maxCount`. 例 `"ABABAB"`, `n = 3` 需要 `(2-1)*4 + 2 = 6`, 只 `+1` 得 5 错.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int leastInterval(vector<char>& tasks, int n) {
            vector<int> freq(26, 0);
            for (char c : tasks) ++freq[c - 'A'];

            int maxFreq  = *max_element(freq.begin(), freq.end());
            int maxCount = count(freq.begin(), freq.end(), maxFreq);   // tie 数

            // 公式 vs 总数, 取大
            return max((int)tasks.size(), (maxFreq - 1) * (n + 1) + maxCount);
        }
    };
    ```

=== "C++ (Heap 版, 需实际序列时用)"
    ```cpp
    class Solution {
    public:
        int leastInterval(vector<char>& tasks, int n) {
            vector<int> freq(26, 0);
            for (char c : tasks) ++freq[c - 'A'];

            priority_queue<int> pq;                    // 大根堆按剩余次数
            for (int f : freq) if (f) pq.push(f);

            int time = 0;
            while (!pq.empty()) {
                vector<int> temp;                      // 本轮 pop 出的暂存
                for (int i = 0; i <= n; i++) {
                    if (!pq.empty()) {
                        int f = pq.top(); pq.pop();
                        if (f > 1) temp.push_back(f - 1);
                    }
                    time++;
                    if (pq.empty() && temp.empty()) break;   // 都空, 不补 idle
                }
                for (int f : temp) pq.push(f);
            }
            return time;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def leastInterval(self, tasks: list[str], n: int) -> int:
            # Counter: dict 子类, 一行完成频次统计, 比手写 for 循环短
            # 相当于 C++ 的 unordered_map<char,int> + for c: cnt[c]++
            freq = Counter(tasks)

            # freq.values() 返频次, max() 取最大
            max_freq = max(freq.values())

            # sum(生成器) 数 tie: Python 无 count-if, 用 sum(bool) 惯用法
            max_count = sum(1 for v in freq.values() if v == max_freq)

            return max(len(tasks), (max_freq - 1) * (n + 1) + max_count)
    ```

=== "JavaScript"
    ```javascript
    var leastInterval = function(tasks, n) {
        // Map 统计频次. Object 也可, 但 Map 迭代顺序更稳定
        // 相当于 C++ 的 unordered_map<char,int>
        const freq = new Map();
        for (const c of tasks) freq.set(c, (freq.get(c) ?? 0) + 1);

        // Math.max(...iterable): 展开语法把 Map values iterator 变参数列表
        // ...values 会把 iterator 展开成 [3, 2, 1] 这样, 再传给 Math.max
        // 相当于 C++ 的 *max_element(freq.begin(), freq.end())
        const maxFreq = Math.max(...freq.values());

        // Array.from(iter).filter(x=>...).length: 收集后过滤计数
        // 相当于 C++ 的 count_if(..., [](int v){ return v == maxFreq; })
        const maxCount = Array.from(freq.values()).filter(v => v === maxFreq).length;

        return Math.max(tasks.length, (maxFreq - 1) * (n + 1) + maxCount);
    };
    ```

## Complexity

| 版本 | Time | Space |
|---|---|---|
| **公式** | **O(n)** | O(1) |
| Heap 模拟 | O(n log 26) ≈ O(n) | O(26) |

## 相关题目

- [0347. Top K Frequent Elements](../../06-stack-queue/0347-top-k-frequent-elements/README.md) — 频次统计 + Top-K
- [0692. Top K Frequent Words](../../06-stack-queue/0692-top-k-frequent-words/README.md) — heap + tie-break
- [0135. Candy](../0135-candy/README.md) — 结构约束 + 双向扫贪心
- [0763. Partition Labels](../0763-partition-labels/README.md) — 用位置骨架分段贪心
- 0767\. Reorganize String (待补) — 直接同款, "字符不相邻" 版本
- 1054\. Distant Barcodes (待补) — heap 模拟排列
- 0358\. Rearrange String k Distance Apart (待补) — n 更一般化
- 1405\. Longest Happy String (待补) — heap + 冷却 3 种字符

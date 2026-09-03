# 0767. Reorganize String / 重构字符串

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Heap, Hash Table, String, Counting · 贪心, 堆, 哈希, 字符串, 计数
    - **Link**: [LeetCode](https://leetcode.com/problems/reorganize-string/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Rearrange so no two adjacent chars are equal**. Feasibility: `maxFreq ≤ ⌈n/2⌉`. Build: **max-heap on frequency**; each round pop the top 2, append both, push each back if remaining. Odd tail gets the last one alone.
>
> **中文**: **重排使相邻字符不同**. 可行性: `maxFreq ≤ ⌈n/2⌉`. 构造: **大根堆按频次**; 每轮 pop 最频的 2 个, 各接一个, 剩余再 push. 末尾单独处理奇数.
>
> *Template / 模版*: **Feasibility check → Greedy heap simulation** — 类同 [0621 Task Scheduler](../0621-task-scheduler/README.md) 的 heap 版, 只是这里是排字符不排调度.

## Problem

**EN**: 重排字符串 `s` 使**相邻字符都不同**. 无解返 `""`.

**中文**: 相邻不重复的重排, 无解返空.

## Key Insights

1. **🔑 灵魂: 可行性判据 — `maxFreq ≤ ⌈n/2⌉` / Feasibility check**

    最频的字符必须"每隔一位摆", 所需槽位:

    ```
    n 偶数: 位 0, 2, 4, ..., n-2  → n/2 个位置
    n 奇数: 位 0, 2, 4, ..., n-1  → (n+1)/2 = ⌈n/2⌉ 个
    ```

    → 若 `maxFreq > ⌈n/2⌉` **一定失败**. Yang 用 `(s.size() + 1) / 2` = 整数除法版 `⌈n/2⌉` (无需 ceil 函数).

    > **`(n + 1) / 2` = `⌈n / 2⌉` (整数除)** — 面试常用招. C++ / Java / JS 全通用.

2. **🔑 灵魂: 为啥 max-heap 每次拿 2 个 / Why pop 2 per round**

    **贪心直觉**: 每步都用**当前最频**的, 但**不能连用同一个** (否则相邻重复). → 拿最频的两个交替.

    - Pop 最频 → append.
    - Pop 次频 → append.
    - 各减 1, 若还有剩再 push 回堆.

    这样保证连续两次 append 的**必是不同字符**. 且每轮消化两个最紧张的, 逼近 balanced.

    > **"每步取 top-2 交替" 是相邻不重复类问题的通用招**. 也用于 [0621](../0621-task-scheduler/README.md), 0358 Rearrange String k Distance, 1054 Distant Barcodes.

3. **🔑 为啥这个贪心正确 / Why the greedy works**

    反证: 若不选**当前最频**, 那它频次只会**继续增长的相对比例**, 后面更难安置. → 每步优先消化最紧张的.

    严格证明用**交换论证**: 假设最优解某步没选 top-2, 交换后合法性不变且解长度不变. → 贪心是最优. (略)

    > **贪心正确性 = 交换论证**. 面试若追问, 就说"proof by exchange argument".

4. **🔑 奇数尾单独处理 / Odd tail: last one alone**

    Yang 代码:

    ```cpp
    while (pq.size() >= 2) { ... 每轮拿 2 个 ... }
    if (!pq.empty()) res += pq.top().second;
    ```

    - 每轮 pop 2, res 长度 +2. 若最终堆里**剩 1 个**, 就是最后一格.
    - **关键**: 此时那唯一剩下的字符**频次一定是 1** — 因为可行性检查通过, 不可能出现 "剩 2 个同字符"; 若真剩 2, 前一轮就该处理它.
    - 若剩下的**频次 > 1** (即 ≥ 2), 说明**可行性检查漏了** — 但 `(n+1)/2` 已经卡住.

    > **易错点 top 1**: 忘处理奇数尾的最后一个字符. 循环用 `>= 2` 而不是 `> 0`.

5. **🔑 用 `pair<int, char>` 让堆按 freq 排序 / Pair for heap ordering**

    `priority_queue<pair<int, char>>` 默认**大根堆按 first**, first 是 freq. → 天然按频次排序.

    - 若用 `pair<char, int>` 就按字符字典序排了 (错).
    - 想稳定 tie-break (如字典序小的先出), 可改 `priority_queue<pair<int, char>, ..., greater<>>` + 反转符号, 或自定义 comparator.

    > **"`pair` 的排序按 first 优先"** 是 STL 常识, 面试用之秒建堆.

6. **🔑 复杂度 / Complexity**

    - **Time**: O(n log k), k = 26 (不同字符数). 每次 pop/push O(log k), 共 n 次 → **O(n log 26) ≈ O(n)**.
    - **Space**: O(k) — 堆最多 k 个元素.

7. **🔑 替代解法: 计数 + 交叉填空 / Counting + interleave (O(n))**

    不用堆也行:

    1. 统计频次, 找 maxFreq 字符 (设为 `x`).
    2. **先把 x 填偶数位** 0, 2, 4, ...
    3. 剩余字符按频次降序 (不严格必需) 填**剩下的偶数位**, 再填**奇数位** 1, 3, 5, ...

    → O(n) 无 heap. 但**heap 版更好写不易错**, 面试推 heap 版.

    > Yang 的 heap 版是标准答案. 想炫技可提 O(n) 交叉填空.

8. **🔑 易错点 top 2 / Pitfalls**

    - **忘处理奇数尾** — 循环用 `size() >= 2`, 结束后检查是否还剩 1 个.
    - **`>` vs `>=` 边界** — 可行性判 `maxFreq > (n+1)/2` (严格大于才失败). `maxFreq == (n+1)/2` 是极限但可行 (奇数尾正好装).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string reorganizeString(string s) {
            unordered_map<char, int> cnt;
            for (char c : s) cnt[c]++;

            priority_queue<pair<int, char>> pq;                    // 大根堆按 freq
            for (auto& [c, freq] : cnt) pq.push({freq, c});

            if (pq.top().first > (int)(s.size() + 1) / 2) return "";   // 可行性

            string res;
            while (pq.size() >= 2) {                               // 每轮 pop 2
                auto [c1, ch1] = pq.top(); pq.pop();
                auto [c2, ch2] = pq.top(); pq.pop();
                res += ch1;
                res += ch2;
                if (--c1) pq.push({c1, ch1});                      // 剩了就回堆
                if (--c2) pq.push({c2, ch2});
            }
            if (!pq.empty()) res += pq.top().second;               // 奇数尾
            return res;
        }
    };
    ```

=== "Python"
    ```python
    import heapq
    from collections import Counter

    class Solution:
        def reorganizeString(self, s: str) -> str:
            # Counter: dict 子类, 一行完成频次
            cnt = Counter(s)

            # Python heapq 默认小根堆. 要大根堆 → freq 取负
            # (-freq, char): 按 -freq 升序 = freq 降序
            # 相当于 C++ 的 priority_queue<pair<int, char>> (大根堆按 first)
            heap = [(-freq, ch) for ch, freq in cnt.items()]
            heapq.heapify(heap)                                    # O(k), 一次性建堆

            # 可行性: 最频字符 ≤ ⌈n/2⌉
            if -heap[0][0] > (len(s) + 1) // 2:
                return ""

            res = []
            while len(heap) >= 2:
                # heappop 弹最小 (即 -freq 最负 = freq 最大)
                f1, ch1 = heapq.heappop(heap)
                f2, ch2 = heapq.heappop(heap)
                res.append(ch1)
                res.append(ch2)
                # freq -1 = (-f) - 1 → -(f+1); 用完还剩则 push 回
                # heappush 也 O(log k)
                if f1 + 1 < 0: heapq.heappush(heap, (f1 + 1, ch1))
                if f2 + 1 < 0: heapq.heappush(heap, (f2 + 1, ch2))

            if heap:
                res.append(heap[0][1])                             # 奇数尾
            # ''.join 比 str += 每步都建新 str 快很多
            return ''.join(res)
    ```

=== "JavaScript"
    ```javascript
    // JS 没有原生优先队列. 面试可以 (a) 手写堆, (b) 用 sort 每轮重排 (n * k log k, k=26 时约等 O(n))
    // 这里选 (b) — 更短更能展示思路

    var reorganizeString = function(s) {
        // Map 计数; 相当于 C++ unordered_map<char,int>
        const cnt = new Map();
        for (const c of s) cnt.set(c, (cnt.get(c) ?? 0) + 1);

        // 转数组 [freq, char], 后续每轮按 freq 降序排
        // Array.from(Map) 返 [[key, val], ...] 展开 iterator
        // 相当于 C++ 的 for (auto& [c, f] : cnt) heap.push({f, c})
        let arr = Array.from(cnt, ([ch, f]) => [f, ch]);

        // 可行性: 最大频 ≤ ⌈n/2⌉
        const maxFreq = Math.max(...arr.map(x => x[0]));
        if (maxFreq > Math.ceil(s.length / 2)) return "";

        const res = [];
        // 循环直到剩 ≤ 1
        while (arr.length >= 2) {
            // 每轮排序 O(k log k), k = 26. 也可用堆库
            arr.sort((a, b) => b[0] - a[0]);
            // 解构赋值 (destructuring) — 类似 Python 的 f, c = arr[0]
            const [f1, ch1] = arr[0];
            const [f2, ch2] = arr[1];
            res.push(ch1, ch2);
            // filter 建新数组比 splice 中间删更安全
            arr[0][0]--; arr[1][0]--;
            arr = arr.filter(x => x[0] > 0);
        }
        if (arr.length) res.push(arr[0][1]);                     // 奇数尾
        return res.join('');
    };
    ```

## Complexity

| 版本 | Time | Space |
|---|---|---|
| **Heap greedy** | **O(n log k)**, k = 26 | O(k) |
| Counting + interleave | O(n) | O(k) |

## 相关题目

- [0621. Task Scheduler](../0621-task-scheduler/README.md) — 相邻不重复的**冷却版**, 姐妹题
- [0347. Top K Frequent Elements](../../06-stack-queue/0347-top-k-frequent-elements/README.md) — 频次 + 堆
- [0692. Top K Frequent Words](../../06-stack-queue/0692-top-k-frequent-words/README.md) — 频次 + 堆 + 字典序
- 0451\. Sort Characters By Frequency (待补) — 频次排序
- 1054\. Distant Barcodes (待补) — 完全同款, 数字版
- 0358\. Rearrange String k Distance Apart (待补) — k 距离间隔一般版
- 1405\. Longest Happy String (待补) — 3 字符 + 冷却 2
- 0984\. String Without AAA or BBB (待补) — 双字符版

# 0692. Top K Frequent Words / 前 K 个高频单词

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Heap, Priority Queue, Custom Comparator, Top-K · 堆, 优先队列, 自定义比较器
    - **Link**: [LeetCode](https://leetcode.com/problems/top-k-frequent-words/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Top K frequent words, ties broken by lexicographic order** → **min-heap of size k** with **custom comparator**: pop the least-frequent, or (on tie) the **lexicographically largest**. Result filled **in reverse** because heap pops from worst to best.
>
> **中文**: **频次前 K, 同频字典序小的优先** → **大小 k 的最小堆** + 自定义 comparator: 弹频次小的, 同频弹字典序**大**的. 结果**倒序**填 (堆先弹最差).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单词数组 `words` + `k`. 返回**频次前 k 个** 单词. **排序**: 频次高在前; **同频次字典序小在前**.

- 例: `["i","love","leetcode","i","love","coding"], k=2` → `["i","love"]` (都出现 2 次, 字典序 `i < love`).

**中文**: 前 k 高频, 同频次字典序小的排前.

## Key Insights

1. **🔑 灵魂: min-heap of k + 自定义 comparator / min-heap of k with custom cmp**

    跟 [0347 Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) 同款"min-heap of size k" 模板 — **但多了 tiebreak**:

    - **不同频**: 频次小的沉堆 (min-heap on count).
    - **同频**: **字典序大** 的沉堆 (让字典序小的留在 top-k 里).

    → **自定义 comparator**.

    > **Top-K 模板 + tiebreak = 自定义比较器**. 见到 tie 立刻反射.

2. **🔑 comparator 反向: `return true ⇒ a 优先级低 ⇒ a 沉底 / Priority queue comparator is *reversed***

    C++ `priority_queue<T, Container, Cmp>`: **`Cmp(a, b) == true` 意思是 "a 优先级比 b 低"** — a 沉下去, 堆顶留下"高优先级".

    Yang 的:

    ```cpp
    auto cmp = [](auto& a, auto& b) {
        if (a.second != b.second) return a.second > b.second;   // a 频次大 → 优先级低 → a 沉底 → min-heap on count
        return a.first < b.first;                                // 同频, a 字典序小 → 优先级低 → a 沉底 → 字典序大的在堆顶
    };
    priority_queue<..., decltype(cmp)> pq(cmp);
    ```

    - **`a.second > b.second`** → a 频次大, a **不该被踢** → 让它优先级低留堆里, 堆顶是 low-count 的 → 一旦超 k 就踢 low-count.
    - **`a.first < b.first`** → 同频次, a 字典序**小** (更优), a 应该留 top-k → **让 a 优先级低留堆里**, 顶是字典序大的.

    → **弹堆顶就是弹"最不够格 top-k 的人"**.

    > **`priority_queue` 的 cmp 语义反直觉**. 记忆: **`return true ⇔ a 沉底 ⇔ b 优先**. 跟 `std::sort` 的 cmp 反向, 是 C++ 的经典坑.

3. **🔑 弹 k 次倒序填 res / Pop k times, fill in reverse**

    ```cpp
    vector<string> res(k);
    for (int i = k - 1; i >= 0; i--) {
        res[i] = pq.top().first;
        pq.pop();
    }
    ```

    **为啥倒序?**

    - min-heap 顶是"堆里最差的" → 先弹出的是**排名末位** (第 k 名).
    - 我们要 res[0] = 第 1 名, res[k-1] = 第 k 名.
    - → 倒着填: `res[k-1]` = 第一次弹的 (最差), `res[0]` = 最后弹的 (最好).

    > **min-heap 出堆顺序天然是"最差 → 最好"**. 记结果时倒填 = 正确排序.

4. **🔑 跟 [0347 Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) 对比 / vs 0347**

    | | 0347 | **0692 (本题)** |
    |---|---|---|
    | 元素 | int | **string** |
    | Tiebreak | 无 | **字典序** |
    | Heap 类型 | `pair<count, value>` 默认 max-heap 可用 | **必须自定义 comparator** |
    | 出堆填 res | 任意顺序 | **倒序** (为保排序) |

    > **一字之差**: 0347 的 `pair<int, int>` 默认排序够用, 本题因**tie 时字典序反向** 必须自定义.

5. **🔑 备选: bucket sort O(n) / Bucket sort alternative**

    - 按 frequency 分桶 (0..n).
    - 每桶内 sort by 字典序.
    - 从高频桶往低频扫, 收集 k 个.

    → **O(n) 时间** 若桶内不排序, **O(n log n) 若排序**. 本题**必须桶内排字典序** → 也是 O(n log n) 但常数更好.

    > 面试 follow-up "能 O(n)?" 若字典序 tie 少可近似.

6. **🔑 复杂度 O(n log k) 时间, O(n + k) 空间 / Complexity**

    - Time: n 次 push/pop 各 O(log k).
    - Space: hash O(n) + heap O(k).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> topKFrequent(vector<string>& words, int k) {
            unordered_map<string, int> freq;
            for (auto& w : words) freq[w]++;

            // min-heap comparator: 频次小或同频字典序大的 = "最不够格 top-k"
            auto cmp = [](const pair<string, int>& a, const pair<string, int>& b) {
                if (a.second != b.second) return a.second > b.second;    // a 频次大 → a 沉底 → 堆顶是低频
                return a.first < b.first;                                 // 同频, a 字典序小 → a 沉底 → 堆顶是字典序大的
            };
            priority_queue<pair<string, int>, vector<pair<string, int>>, decltype(cmp)> pq(cmp);

            for (auto& [w, c] : freq) {
                pq.push({w, c});
                if ((int)pq.size() > k) pq.pop();                        // 踢最差的
            }

            vector<string> res(k);
            for (int i = k - 1; i >= 0; i--) {                            // 倒序填
                res[i] = pq.top().first;
                pq.pop();
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    import heapq
    from collections import Counter

    class Solution:
        def topKFrequent(self, words: list[str], k: int) -> list[str]:
            freq = Counter(words)
            # Python heapq 是 min-heap. 用 (-count, word) 让 count 大的排前
            # 同 count 时 word 字典序小的排前 — 天然 min-heap on tuple 就是这个语义
            # 直接 heapq.nsmallest(k, key=...) 更 Pythonic:
            return heapq.nsmallest(k, freq.keys(), key=lambda w: (-freq[w], w))

        # 手写 heap of k 版 (对齐 C++ 教学)
        def topKFrequent_heap(self, words: list[str], k: int) -> list[str]:
            freq = Counter(words)
            heap = []                       # (freq, word) — 想 min-heap on freq, max-heap on word (同频)
            for w, c in freq.items():
                # (c, w): min-heap 顶就是 low-count; 同 c 时字典序小的顶
                # 但我们要**弹字典序大的** 保留字典序小 → 存 (c, negated word)? 字符串没直接取负
                # 用 wrapper: 存 (c, w), 手写 cmp — Python 用 heapq 时不能自定义 cmp, 只能靠 key 转换
                # 巧: 存 (-c, w) 是错的 (因为 -c 让 max-count 顶, 但我们要 min-count 顶)
                # 解决: 用 kick 逻辑 — 存 (c, w), 但弹的时候看堆顶是否"该弹" (即 count 小 or 同 count word 大)
                # → 这不好写. 更直接: 用 heapq.nsmallest 上面那行.
                heapq.heappush(heap, (c, [-ord(ch) for ch in w], w))    # tricky, 建议直接用 nsmallest
                if len(heap) > k:
                    heapq.heappop(heap)
            heap.sort(key=lambda x: (-x[0], x[2]))
            return [x[2] for x in heap]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} words
     * @param {number} k
     * @return {string[]}
     */
    var topKFrequent = function(words, k) {
        // JS 无原生 heap. 简易: 排序 + 取前 k
        // 面试严谨要手写 min-heap of k
        const freq = new Map();
        for (const w of words) freq.set(w, (freq.get(w) || 0) + 1);
        // 排序 cmp: 频次降序; 同频字典序升序
        return [...freq.keys()]
            .sort((a, b) => freq.get(b) - freq.get(a) || a.localeCompare(b))
            .slice(0, k);
    };
    ```

## Complexity

- **Time**: O(n log k) — n 次 push/pop 各 O(log k).
- **Space**: O(n + k) — hash + heap.

## 相关题目

- [0347. Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) — 频次 Top-K 无 tiebreak
- [0215. Kth Largest Element in an Array](../0215-kth-largest-element-in-an-array/README.md) — Kth 单值
- [0973. K Closest Points to Origin](../0973-k-closest-points-to-origin/README.md) — 距离 Top-K
- [0703. Kth Largest Element in a Stream](../0703-kth-largest-element-in-a-stream/README.md) — 流式版
- [1046. Last Stone Weight](../1046-last-stone-weight/README.md) — max-heap 模拟
- [0239. Sliding Window Maximum](../0239-sliding-window-maximum/README.md) — 单调队列
- 0451\. Sort Characters By Frequency (待补) — 字符频次排序
- 0451\. Sort Characters By Frequency (待补) — bucket sort 变体
- 1636\. Sort Array by Increasing Frequency (待补) — 类似 tiebreak 排序

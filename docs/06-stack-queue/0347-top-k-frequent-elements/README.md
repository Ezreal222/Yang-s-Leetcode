# 0347. Top K Frequent Elements / 前 K 个高频元素

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Heap, Hash Map, Bucket Sort, Priority Queue · 堆, 哈希表, 桶排序, 优先队列
    - **Link**: [LeetCode](https://leetcode.com/problems/top-k-frequent-elements/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☑ ☐

## TL;DR / 一句话

> **EN**: **Find k most frequent** → step 1: hash count. Step 2 pick one: **bucket sort** (O(n), 频次 ≤ n 时最优) / **min-heap of size k** (O(n log k), k ≪ n 时最优) / max-heap pop k (lazy 写法).
>
> **中文**: **取前 k 高频** → 先哈希计数, 再三选一: **桶排** (O(n), 频次封顶时最快) / **大小 k 的最小堆** (O(n log k), 流式友好) / max-heap 弹 k 次 (最懒).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给整数数组和 `k`, 返回出现次数前 `k` 高的元素 (顺序无所谓).

**中文**: 数组里出现次数最多的前 k 个元素.

## 思路

Step 1 都一样: `unordered_map<int,int>` / `Counter` 计频次. Step 2 三选一:

| 思路 | Time | Space | 适用 |
|---|---|---|---|
| A. Max-heap, push 全部, 弹 k 次 | O(n log n) | O(n) | 写起来最简单 |
| B. **Min-heap of size k** (经典 Top-K) | **O(n log k)** | **O(k)** | k ≪ n, 流式数据 |
| C. **Bucket sort** (按频次分桶) | **O(n)** | O(n) | 频次有上界 (=n), 性能上限 |

> 教科书答案是 **min-heap of size k** —— 维护大小 k 的最小堆, 新元素跟堆顶比, 大就替换. 堆里最后剩的就是 Top-K. **Bucket sort** 是 O(n) 最优, 但只在频次有自然上界时好用 (这题正好: 频次最大就是 n).

### Heap cheat sheet / 堆速查

**C++** `priority_queue`:

| | declaration | top is |
|---|---|---|
| max-heap (default) | `priority_queue<T> pq;` | largest |
| min-heap | `priority_queue<T, vector<T>, greater<T>> pq;` | smallest |
| custom (struct functor) | `priority_queue<T, vector<T>, Cmp> pq;` | depends on `Cmp` |

Comparator 反着来 —— `Cmp(a,b)` 返 `true` 表示 "a 优先级**比 b 低**" → a 沉下去. 想 min-heap on `.second`, 写 `a.second > b.second` (大的沉, 小的浮).

`pair<int,int>` 默认按 `first` 升序 → `second` 升序排. 配合 `priority_queue<pair<int,int>>` (max-heap) 就是按 `first` 降序优先. 所以 **Variant A 把 `(count, value)` 压进去** → `top().second` 就是当前最高频的 value.

**Python** `heapq`:

- 默认 **min-heap**. 想 max-heap → 存 `-x` 或 `(-key, x)`.
- 直接拿 Top-K 的两个利器:
    - `heapq.nlargest(k, iterable, key=...)` — O(n log k), 内部就是大小 k 的 min-heap.
    - `collections.Counter(...).most_common(k)` — 一行解决.

### Pattern: min-heap of size k for Top-K largest

通用模板, 解一类题 (Top K Frequent / Closest / Stocks / …):

```text
for each item x:
    push x into heap
    if heap.size() > k:
        heap.pop()      # pops the SMALLEST → kicks the loser
return heap contents    # the k largest
```

> **为啥是 min-heap 不是 max-heap?** 我们想**只留前 k 大**, 所以堆里要能快速找"目前最差的一个" 把它踢掉. min-heap 顶就是堆里最小, O(1) 查, O(log k) 踢.

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to return the **k most frequent elements** in `nums`. A few things to confirm:"

- "**Return order matters?** Do they need to be sorted by frequency, or is any order fine?" *(usually any order — that unlocks options.)*
- "**Guaranteed unique answer?** i.e., no ties at rank k that force a choice?" *(LC guarantees the answer is unique. If ties mattered, we'd need a tiebreaker like [0692 Top K Frequent Words](../0692-top-k-frequent-words/README.md).)*
- "Bound on n?" *(up to ~10⁵ — need at least O(n log k), ideally O(n).)*
- "**k vs n?** Any guarantee `k ≤ number of unique elements`?" *(yes.)*

### 2. Brainstorm approaches (1 min)

> "Step 1 is always the same: **count frequencies with a hash map** — O(n). Then step 2 is where the choice matters. Three options:
>
> **Variant A — sort by count, take first k**: O(n log n). Simplest, but sorts more than needed.
>
> **Variant B — min-heap of size k**: iterate through frequency entries, push each into a min-heap keyed by count. When size exceeds k, pop — that kicks the least frequent. At the end the heap contains the top k. **O(n log k)**, O(k) space. This is the streaming-friendly classical answer.
>
> **Variant C — bucket sort**: since counts are bounded by n, allocate `buckets[0..n]` where `buckets[c]` holds all values with frequency c. Walk buckets from high to low, collect k. **O(n)** — asymptotically optimal, works because 'count is bounded by input size' gives us a natural finite bucket space.
>
> I'd lead with **Variant B** (heap) — most robust template. If they want optimal, I'll do **Variant C**."

### 3. Sketch the algorithm before coding (1 min)

> "For the heap approach, two design details worth calling out:
>
> 1. **Min-heap, not max-heap.** Sounds counterintuitive — we want the *most* frequent, so why the *min* heap? Because we're maintaining 'the top k so far' as a set, and to keep size ≤ k, we need to **kick out the worst** on overflow. 'The worst' among current top-k is the **least frequent** — a min-heap gives us O(1) access to that.
> 2. **Heap element is `(count, value)`.** Frequency is the sort key, so it goes first. In C++ `pair<int, int>` sorts by first lexicographically, which matches.
>
> The loop: for each `(value, count)` in the freq map — push `(count, value)`, then if heap size > k, pop. At the end, the heap contains exactly the k most frequent values."

### 4. Code while narrating (2 min)

```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int, int> freq;
    for (int x : nums) freq[x]++;

    // min-heap of (count, value)
    priority_queue<pair<int, int>,
                   vector<pair<int, int>>,
                   greater<>> pq;
    for (auto& [val, cnt] : freq) {
        pq.push({cnt, val});
        if ((int)pq.size() > k) pq.pop();
    }
    vector<int> res;
    while (!pq.empty()) {
        res.push_back(pq.top().second);
        pq.pop();
    }
    return res;
}
```

> "About 15 lines. The `greater<>` template makes it a min-heap. The push-then-maybe-pop pattern is the top-K workhorse."

### 5. Trace an example (1 min)

> "Let me trace `nums = [1,1,1,2,2,3], k = 2`:
>
> - After counting: `freq = {1:3, 2:2, 3:1}`.
> - Iterate (order doesn't matter — say `(1,3), (2,2), (3,1)`):
>   - Push `(3, 1)` → heap `[(3, 1)]`. Size 1 ≤ 2.
>   - Push `(2, 2)` → heap `[(2, 2), (3, 1)]` (min-heap orders by first). Size 2 ≤ 2.
>   - Push `(1, 3)` → heap `[(1, 3), (3, 1), (2, 2)]`. Size 3 > 2 → pop `(1, 3)` (lowest count kicked). Heap `[(2, 2), (3, 1)]`.
> - Extract: `[2, 1]`. ✓ These are the two most frequent — 1 (count 3) and 2 (count 2). Order doesn't matter."

### 6. Complexity (20s)

> "**Time O(n log k)** — n frequency entries × O(log k) per heap op. **Space O(n + k)** — n for the frequency map, k for the heap.
>
> Bucket sort variant is **O(n)** time, O(n) space — better when we want the absolute optimum, and it's not much more code."

### 7. Follow-ups + related (1 min)

> "A few directions:
>
> 1. **Bucket sort for optimal O(n)**: `vector<vector<int>> buckets(n+1)`; for each `(val, cnt)`, put val in `buckets[cnt]`. Walk from `buckets[n]` down, collect until we have k. Simple, no heap.
>
> 2. **Streaming version**: if elements arrive one at a time and we want top-k on demand, the min-heap adapts naturally — same push-then-maybe-pop.
>
> 3. **Add a tiebreaker** (like [0692 Top K Frequent Words](../0692-top-k-frequent-words/README.md)): switch to a custom comparator so same-count elements sort by lex order (or whatever the tie rule is). The push-pop template stays.
>
> 4. **Quickselect** for a Kth Element flavor ([0215 Kth Largest](../0215-kth-largest-element-in-an-array/README.md)): expected O(n), but doesn't fit as cleanly when we need the top *set*, not just a rank.
>
> Any follow-ups you'd like me to code?"

## Solution

### Variant A — max-heap, pop k 次

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            unordered_map<int, int> m;
            for (int n : nums) m[n]++;
            // pair<count, value>, max-heap by .first (默认)
            priority_queue<pair<int, int>> pq;
            for (auto& [val, cnt] : m) pq.push({cnt, val});
            vector<int> res;
            for (int i = 0; i < k; i++) {
                res.push_back(pq.top().second);
                pq.pop();
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            # most_common(k) 内部就是 nlargest(k, ...) — 即"min-heap of size k"
            # C++ 等价: 手搓的 Variant B. Python 一行写完
            return [v for v, _ in Counter(nums).most_common(k)]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number[]}
     */
    var topKFrequent = function(nums, k) {
        // 1) 用 Map 计频次. Map 的 key 接受数字, 比 Object 干净 (Object key 都被转 string)
        const freq = new Map();
        for (const x of nums) freq.set(x, (freq.get(x) || 0) + 1);
        // 2) JS 没原生 priority_queue. 偷懒: 把 [value, count] 全排, 取前 k
        // .sort((a, b) => b[1] - a[1]) 按频次降序. O(n log n) — Variant A 同阶
        return [...freq.entries()]
            .sort((a, b) => b[1] - a[1])
            .slice(0, k)
            .map(([v]) => v);
    };
    ```

### Variant B — min-heap of size k (the canonical Top-K pattern)

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            unordered_map<int, int> freq;
            for (int x : nums) freq[x]++;
            // (count, value), 用 greater<> 直接得到 min-heap on .first
            // 比写自定义 Compare 类干净一截
            priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;
            for (auto& [num, f] : freq) {
                pq.push({f, num});
                if ((int)pq.size() > k) pq.pop();        // 超 k 就踢掉频次最小的
            }
            vector<int> ans;
            while (!pq.empty()) {
                ans.push_back(pq.top().second);
                pq.pop();
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter
    import heapq

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            cnt = Counter(nums)
            heap: list[tuple[int, int]] = []        # (count, value); heapq 默认按元组首位 min-heap
            for v, c in cnt.items():
                heapq.heappush(heap, (c, v))
                if len(heap) > k:
                    heapq.heappop(heap)             # 弹掉 count 最小的
            return [v for _, v in heap]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number[]}
     */
    var topKFrequent = function(nums, k) {
        // JS 标准库无堆. 真要写 min-heap of k 得手搓 — 这里给 Variant C (桶排) 的 JS 版,
        // 它在 JS 里反而更实用. 见下面 Variant C.
        // (这里复用 Variant A 写法, 因为 JS 没堆)
        const freq = new Map();
        for (const x of nums) freq.set(x, (freq.get(x) || 0) + 1);
        return [...freq.entries()]
            .sort((a, b) => b[1] - a[1])
            .slice(0, k)
            .map(([v]) => v);
    };
    ```

### Variant C — bucket sort (O(n))

频次最大就 `n` → 开 `n + 1` 个桶, `buckets[freq]` 装所有出现 `freq` 次的值. 从高频桶往低频扫, 收齐 k 个就停.

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            int n = nums.size();
            unordered_map<int, int> freq;
            for (int x : nums) freq[x]++;
            vector<vector<int>> buckets(n + 1);                          // buckets[i] = 频次为 i 的元素
            for (auto& [num, f] : freq) buckets[f].push_back(num);
            vector<int> ans;
            for (int i = n; i >= 1 && (int)ans.size() < k; i--) {        // 从高频往低频
                for (int num : buckets[i]) {
                    ans.push_back(num);
                    if ((int)ans.size() == k) break;
                }
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            cnt = Counter(nums)
            # buckets[i] = 出现 i 次的所有元素. 列表推导式 [[] for _ in ...] 是必须的,
            # 不能用 [[]] * (n+1) — 那会让所有桶共享同一个 list 引用!
            buckets: list[list[int]] = [[] for _ in range(len(nums) + 1)]
            for v, c in cnt.items():
                buckets[c].append(v)
            res: list[int] = []
            for i in range(len(buckets) - 1, 0, -1):    # 从大到小遍历桶
                res.extend(buckets[i])
                if len(res) >= k:
                    return res[:k]
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number[]}
     */
    var topKFrequent = function(nums, k) {
        const n = nums.length;
        const freq = new Map();
        for (const x of nums) freq.set(x, (freq.get(x) || 0) + 1);
        // Array.from + 工厂函数, 不能 new Array(n+1).fill([]) — fill 会让所有槽指向同一个数组
        const buckets = Array.from({length: n + 1}, () => []);
        for (const [v, c] of freq) buckets[c].push(v);
        const ans = [];
        for (let i = n; i >= 1 && ans.length < k; i--) {
            for (const v of buckets[i]) {
                ans.push(v);
                if (ans.length === k) break;
            }
        }
        return ans;
    };
    ```

## Complexity

| Variant | Time | Space |
|---|---|---|
| A. max-heap | O(n log n) | O(n) |
| B. min-heap of k | **O(n log k)** | **O(k)** |
| C. bucket sort | **O(n)** | O(n) |

A 写起来最快, B 是 Top-K 的通用招式 (适合流式 / k 远小于 n), C 在频次有上界时性能上限.

## 易错点

- **C++ comparator 反着写**: `priority_queue` 的 `Cmp(a,b) == true` 意思是 "**a 优先级低**", a 沉. 想 min-heap 写 `a > b`, 想 max-heap 写 `a < b`. 这和 `std::sort` 的语义相反, 翻车率高. 用 `greater<>` 模板能避开自写 comparator.
- **Min-heap of size k 的 pop 时机**: 必须**每次 push 之后立刻** 判 `size > k` 再 pop —— 不能等都装完再统一处理, 否则就是 O(n log n) 没节省到.

## 相关题目

- [0239. Sliding Window Maximum / 滑动窗口最大值](../0239-sliding-window-maximum/README.md) — 单调队列, 另一种"维护极值"
- [0703. Kth Largest Element in a Stream](../0703-kth-largest-element-in-a-stream/README.md) — 流式版, min-heap of size k 直接应用
- [0215. Kth Largest Element in an Array](../0215-kth-largest-element-in-an-array/README.md) — Top-K 单值, min-heap + quickselect 两版
- [0973. K Closest Points to Origin](../0973-k-closest-points-to-origin/README.md) — 同款 heap-of-k, key = 距离²
- 0451\. Sort Characters By Frequency (待补) — 桶排的另一道
- [0692. Top K Frequent Words](../0692-top-k-frequent-words/README.md) — 加字典序 tiebreak 的变体
- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 计数数组基础
- [0049. Group Anagrams](../../03-hash-table/0049-group-anagrams/README.md) — 哈希分桶模式

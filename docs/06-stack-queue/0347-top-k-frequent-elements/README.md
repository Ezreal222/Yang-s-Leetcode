# 0347. Top K Frequent Elements / 前 K 个高频元素

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Heap, Hash Map, Bucket Sort, Priority Queue · 堆, 哈希表, 桶排序, 优先队列
    - **Link**: [LeetCode](https://leetcode.com/problems/top-k-frequent-elements/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem / 题意

**EN**: Given an array of integers and `k`, return the `k` most frequent values. Order of output doesn't matter.

**中文**: 给数组和 `k`, 返回出现次数前 `k` 高的元素 (顺序无所谓).

## Approach / 思路

Step 1 同款: 用 `unordered_map<int,int>` / `Counter` 数频次. 之后选三种之一:

| 思路 | Time | Space | 适用 |
|---|---|---|---|
| A. Max-heap, push 全部, 弹 k 次 | O(n log n) | O(n) | 写起来最简单, k 大也能用 |
| B. **Min-heap of size k** (经典 Top-K) | O(n log k) | O(k) | k ≪ n 时最划算, 流式数据也能扛 |
| C. **Bucket sort** (按频次分桶) | O(n) | O(n) | 频次有上界 (=n), 性能上限 |

EN takeaways: Top-K 的"教科书"答案是 **min-heap of size k** —— 维护一个大小为 k 的最小堆, 新元素来了和堆顶比, 比堆顶大就替换. 堆里最后剩下的就是 Top-K. Bucket sort 是 O(n) 最优, 但只在频次有自然上界时好用 (这题正好: 频次最大就是 n).

### Heap cheat sheet / 堆速查

**C++** `priority_queue`:

| | declaration | top is |
|---|---|---|
| max-heap (default) | `priority_queue<T> pq;` | largest |
| min-heap | `priority_queue<T, vector<T>, greater<T>> pq;` | smallest |
| custom (struct functor) | `priority_queue<T, vector<T>, Cmp> pq;` | depends on `Cmp` |

Comparator 是反着来的 —— `Cmp(a,b)` 返回 `true` 表示 "a 优先级**比 b 低**" → a 沉到下面. 想要 min-heap on `.second`, 写 `a.second > b.second` (大的沉下去, 小的浮上来).

`pair<int,int>` 默认按 `first` 升序、`second` 升序排. 配合 `priority_queue<pair<int,int>>` (max-heap) 就是按 `first` 降序优先 —— 所以 Variant A 把 `(count, value)` 这样压进去, `top().second` 就是当前最高频的 value.

**Python** `heapq`:

- 默认 **min-heap**. `heapq.heappush(h, x)`, `heapq.heappop(h)`.
- 想要 max-heap: 存 `-x` (数字) 或 `(-key, x)` (元组).
- 直接拿 Top-K 的两个利器:
    - `heapq.nlargest(k, iterable, key=...)` —— O(n log k), 内部就是用大小 k 的 min-heap.
    - `collections.Counter(...).most_common(k)` —— 一行解决这题. 内部也是 nlargest.

### Pattern: min-heap of size k for Top-K largest

通用模板, 解一类题 (Top K Frequent / Closest / Stocks / etc.):

```text
for each item x:
    push x into heap
    if heap.size() > k:
        heap.pop()      # pops the SMALLEST → kicks the loser
return heap contents    # the k largest
```

为啥 min-heap 而不是 max-heap? 因为我们想**只保留前 k 个最大的**, 所以堆里要能快速找到"目前最差的一个"(也就是当前堆里最小的) 把它踢掉. min-heap 顶就是堆里最小, 一查 O(1), 一踢 O(log k).

## Solution / 题解

### Variant A — max-heap, pop k 次

=== "Python"
    ```python
    from collections import Counter
    import heapq

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            cnt = Counter(nums)
            # nlargest 内部就是 min-heap of size k —— 直接用就好
            return [v for v, _ in cnt.most_common(k)]
    ```

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            unordered_map<int, int> m;
            for (int n : nums) m[n]++;
            // pair<count,value>, max-heap by .first (default)
            priority_queue<pair<int, int>> pq;
            for (auto& it : m) pq.push({it.second, it.first});
            vector<int> res;
            for (int i = 0; i < k; i++) {
                res.push_back(pq.top().second);
                pq.pop();
            }
            return res;
        }
    };
    ```

=== "Java"
    ```java
    // TBD
    ```

### Variant B — min-heap of size k (the canonical Top-K pattern)

=== "Python"
    ```python
    from collections import Counter
    import heapq

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            cnt = Counter(nums)
            heap: list[tuple[int, int]] = []   # (count, value); min-heap on count
            for v, c in cnt.items():
                heapq.heappush(heap, (c, v))
                if len(heap) > k:
                    heapq.heappop(heap)        # kick the smallest count
            return [v for _, v in heap]
    ```

=== "C++"
    ```cpp
    class Solution {
        // Min-heap on count: 让"频次最小"那个浮到堆顶, 方便踢
        struct Compare {
            bool operator()(const pair<int, int>& a, const pair<int, int>& b) {
                return a.second > b.second;  // a 频次更高 → 优先级更低 → 沉下去
            }
        };
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            unordered_map<int, int> m;
            for (int n : nums) m[n]++;
            priority_queue<pair<int, int>, vector<pair<int,int>>, Compare> pq;
            for (auto& p : m) {
                pq.push(p);                              // (value, count)
                if ((int)pq.size() > k) pq.pop();        // 一旦超过 k, 弹掉最小频次
            }
            vector<int> res(k);
            // 倒着填: 堆是 min on count, 弹出顺序是从小到大,
            // 但题目不要求有序, 这里倒填只是和 Carl 风格对齐
            for (int i = k - 1; i >= 0; i--) {
                res[i] = pq.top().first;
                pq.pop();
            }
            return res;
        }
    };
    ```

=== "Java"
    ```java
    // TBD
    ```

### Variant C — bucket sort (O(n))

频次最大就 `n`, 所以开 `n+1` 个桶, `buckets[freq]` 装所有出现 `freq` 次的值. 从高频桶往低频扫, 收齐 k 个就停.

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def topKFrequent(self, nums: list[int], k: int) -> list[int]:
            cnt = Counter(nums)
            # buckets[i] = values that occur exactly i times
            buckets: list[list[int]] = [[] for _ in range(len(nums) + 1)]
            for v, c in cnt.items():
                buckets[c].append(v)
            res: list[int] = []
            for i in range(len(buckets) - 1, 0, -1):
                res.extend(buckets[i])
                if len(res) >= k:
                    return res[:k]
            return res
    ```

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> topKFrequent(vector<int>& nums, int k) {
            unordered_map<int, int> counts;
            int max_count = 0;
            for (int n : nums) {
                max_count = max(max_count, ++counts[n]);  // 顺手记录最大频次, 桶大小够用就行
            }
            vector<vector<int>> buckets(max_count + 1);
            for (const auto& p : counts) buckets[p.second].push_back(p.first);
            vector<int> res;
            // 从最高频桶往下扫, 收齐 k 个就停
            for (int i = max_count; i >= 0 && (int)res.size() < k; i--) {
                for (int n : buckets[i]) {
                    res.push_back(n);
                    if ((int)res.size() == k) break;
                }
            }
            return res;
        }
    };
    ```

=== "Java"
    ```java
    // TBD
    ```

## Complexity / 复杂度

| Variant | Time | Space |
|---|---|---|
| A. max-heap | O(n log n) | O(n) |
| B. min-heap of k | **O(n log k)** | **O(k)** |
| C. bucket sort | **O(n)** | O(n) |

A 写起来最快, B 是 Top-K 的通用招式 (适合流式 / k 远小于 n), C 在频次有上界时性能上限.

## Pitfalls / 易错点

- **C++ comparator 反着写**: `priority_queue` 的 `Cmp(a,b) == true` 意思是 "**a 优先级低**", a 沉. 想 min-heap 写 `a > b`, 想 max-heap 写 `a < b`. 这和 `std::sort` 的语义相反, 翻车率高.
- **`pair` 的默认顺序**: `priority_queue<pair<int,int>>` (无 comparator) 是按 `.first` 降序、再 `.second` 降序的 max-heap. Variant A 利用了这点把 `(count, value)` 直接压进去 → top 就是当前最高频. 如果是 `(value, count)` 顺序就反了.
- **Min-heap of size k 的 Pop 时机**: 必须在每次 push 之后立刻判 `size > k` 再 pop —— 不能等都装完再统一处理, 否则就成 O(n log n) 没节省.
- **Bucket sort 的桶大小**: 用 `nums.size() + 1` 就稳 (频次最大就是 n). 用 `max_count + 1` 节省内存但要先扫一遍记录, 看哪种更顺手.
- **Python `heapq` 没有 max-heap**: 想要 max-heap 把值取负 (`heapq.heappush(h, -x)`) 或包成 `(-key, item)`. 但这题用 `nlargest` / `most_common` 直接就过了, 不用手搓.
- **`Counter.most_common(k)` 是这题最短解**: 一行. 面试的时候手写要有, 但实际工程代码就是这一行.

## Related / 相关题目

- [0239. Sliding Window Maximum / 滑动窗口最大值](../0239-sliding-window-maximum/README.md) — 单调队列, 另一种"维护极值"的招式
- 0703. Kth Largest Element in a Stream (待补) — 流式版本, min-heap of size k 的最直接应用
- 0215. Kth Largest Element in an Array (待补) — Top-K 单值, 也可用 quickselect O(n)
- 0973. K Closest Points to Origin (待补) — 同样的 min-heap-of-k 套路, 改个 key
- 0451. Sort Characters By Frequency (待补) — 桶排序的另一道

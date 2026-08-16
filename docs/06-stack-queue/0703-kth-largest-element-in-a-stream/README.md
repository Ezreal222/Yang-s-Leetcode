# 0703. Kth Largest Element in a Stream / 数据流中的第 K 大元素

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Heap, Priority Queue, Design, Streaming · 堆, 优先队列, 设计, 流式
    - **Link**: [LeetCode](https://leetcode.com/problems/kth-largest-element-in-a-stream/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Support `add(val)` returning the k-th largest so far** → **min-heap of size k**: push, pop if size > k. Heap top = kth largest. O(log k) per add.
>
> **中文**: **流式 `add(val)` 返当前第 k 大** → **大小 k 的最小堆**: push, size > k 就 pop. 堆顶 = 第 k 大. 每次 add O(log k).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 设计一个类支持:

- `KthLargest(k, nums)` — 初始化.
- `int add(int val)` — 加入 val, 返回**当前**第 k 大的元素.

**中文**: 流式维护第 k 大.

## Key Insights

1. **🔑 灵魂: 只维护"目前最大的 k 个" → min-heap of size k / Maintain top-k only**

    我们**不关心** 数据流里所有元素, 只关心"**目前最大的 k 个**". **min-heap of size k** 是天然结构:

    - **堆里的 k 个**就是目前最大的 k 个.
    - **堆顶** (最小的那个) 就是**第 k 大**.

    每次 add(val):

    - **push val** into heap.
    - **若 size > k**, **pop** — 弹掉的一定是堆里最小的 (即"被踢出 top-k 的人").
    - **返 top**.

    > **Top-K 流式母题**. 跟 [0215 Kth Largest](../0215-kth-largest-element-in-an-array/README.md) v1 / [0347 Top-K Frequent](../0347-top-k-frequent-elements/README.md) / [0973 K Closest](../0973-k-closest-points-to-origin/README.md) 同款模板.

2. **🔑 为啥 min-heap 而非 max-heap? / Why min-heap, not max-heap**

    要"目前最大的 k 个", 用 **min-heap** 因为**要踢的是"目前最小的那个"** (那个"最不够格 top-k 的人"). min-heap 顶就是最小, O(1) 查, O(log k) 踢.

    若用 max-heap, 堆顶是最大 — 踢它反而**留下小的**, 逻辑相反.

    > **踢什么 → 用什么堆**: 踢最小用 min, 踢最大用 max. 记牢方向.

3. **🔑 构造函数复用 `add` — DRY / Constructor reuses add**

    Yang 的巧:

    ```cpp
    KthLargest(int k, vector<int>& nums) : k(k) {
        for (int x : nums) add(x);      // 复用 add
    }
    ```

    构造函数**不重写** 处理逻辑, 直接**一个个 add**. 跟 [0707 Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) 里 `addHead/Tail` 转 `addAtIndex` 同款 DRY 思维.

    > **"多个 API 共享同一份逻辑"** — 少一半 bug 机会, 代码维护也简单.

4. **🔑 `add` 的 3 行核心 / add's 3-line core**

    ```cpp
    int add(int val) {
        pq.push(val);
        if (pq.size() > k) pq.pop();
        return pq.top();
    }
    ```

    - **push**: O(log k) 因为堆最多 k+1 个元素.
    - **可能 pop**: O(log k).
    - **top**: O(1).
    - **总 O(log k)** 每次 add.

    > 3 行代码, 深度 O(log k). 是**极简 + 高效**的完美设计.

5. **🔑 跟 [0215 静态版](../0215-kth-largest-element-in-an-array/README.md) 对比 / vs 0215 (batch version)**

    | | 0215 静态 | **0703 (本题, 流式)** |
    |---|---|---|
    | 输入 | 一次给全 | **一个个进来** |
    | 查询 | 一次求第 k 大 | **每次 add 都要返** |
    | 最优 | quickselect O(n) | **min-heap of k O(log k) per add** |
    | 为啥不能 quickselect? | — | 无法边插入边 partition |

    > **"流式"** 是把 heap 从"可选" 升级到"必选" — 因为 quickselect 天生不适合动态数据.

6. **🔑 heap 里的其他 k-1 个元素永远无用? / Wait — we only use top!**

    对! 除了堆顶 (第 k 大), 剩下的 k-1 个我们**不查询**. 但它们**必须留着**, 因为:

    - 未来 add 一个**更大** 的值时, 顶部会被换下, 我们需要**下一个候选** → 就是堆里第二小.
    - 若只存"当前第 k 大", 下次挤下去后就没备胎, 得**重新扫**.

    > **"备胎数据" 的存在** 是流式数据结构的常见模式. 如 0295 双堆维中位数 存两半, 每半都是备胎.

7. **🔑 复杂度 / Complexity**

    | 操作 | Time | Space |
    |---|---|---|
    | 构造 | O(n log k) — n 次 add | O(k) |
    | **`add`** | **O(log k)** | O(1) per call |
    | **`top`** (隐含返值) | **O(1)** | — |

    > 若不 cap 堆大小 (全存进去), 每次 add O(log n) 查第 k 大要额外操作. **cap 在 k** 才是关键优化.

## Solution

=== "C++"
    ```cpp
    class KthLargest {
    public:
        priority_queue<int, vector<int>, greater<int>> pq;      // min-heap
        int k;
        KthLargest(int k, vector<int>& nums) : k(k) {
            for (int x : nums) add(x);                          // 复用 add
        }
        int add(int val) {
            pq.push(val);
            if ((int)pq.size() > k) pq.pop();                   // 超 k 就踢最小
            return pq.top();                                    // 堆顶 = 第 k 大
        }
    };
    ```

=== "Python"
    ```python
    import heapq

    class KthLargest:
        def __init__(self, k: int, nums: list[int]):
            self.k = k
            # heapq 是 min-heap — 天然适合本题
            # 先把 nums 全 push 进 heap, 再截到 k (nlargest 更简洁)
            self.heap = nums[:]         # 拷贝防污染
            heapq.heapify(self.heap)    # O(n) 原地建堆
            # 缩到 k 个: 弹掉多余的小的
            while len(self.heap) > k:
                heapq.heappop(self.heap)

        def add(self, val: int) -> int:
            # heappushpop: push + pop 一步, 常数比分开写快
            # 只在 val > 堆顶时才实际替换; ≤ 堆顶时直接返当前 top
            if len(self.heap) < self.k:
                heapq.heappush(self.heap, val)
            elif val > self.heap[0]:
                heapq.heapreplace(self.heap, val)   # pop 顶 + push val, 一步
            return self.heap[0]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} k
     * @param {number[]} nums
     */
    var KthLargest = function(k, nums) {
        // JS 无原生 heap. 简易实现: 每次 add 后 sort, O(n log n) per add
        // 严谨面试要手写 min-heap 或用 priority-queue lib
        this.k = k;
        this.nums = [...nums].sort((a, b) => a - b);    // 升序
        // 截到最后 k 个 (最大的 k 个)
        if (this.nums.length > k) this.nums = this.nums.slice(-k);
    };

    /**
     * @param {number} val
     * @return {number}
     */
    KthLargest.prototype.add = function(val) {
        // 二分插入维持升序 — 手写更严谨
        let lo = 0, hi = this.nums.length;
        while (lo < hi) {
            const mid = (lo + hi) >> 1;
            if (this.nums[mid] < val) lo = mid + 1;
            else hi = mid;
        }
        this.nums.splice(lo, 0, val);
        // 保持只留最后 k 个
        if (this.nums.length > this.k) this.nums.shift();
        return this.nums[0];    // 第 k 大 = 数组首 (剩下最大的 k 个里最小的)
    };
    ```

## Complexity

| 操作 | Time | Space |
|---|---|---|
| 构造 | O(n log k) | O(k) |
| `add` | **O(log k)** | — |

## 相关题目

- [0215. Kth Largest Element in an Array](../0215-kth-largest-element-in-an-array/README.md) — 静态版, min-heap + quickselect
- [0347. Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) — 频次 Top-K, 同款 heap-of-k
- [0973. K Closest Points to Origin](../0973-k-closest-points-to-origin/README.md) — max-heap of k, dist² 优化
- [1046. Last Stone Weight](../1046-last-stone-weight/README.md) — max-heap 模拟
- [0239. Sliding Window Maximum](../0239-sliding-window-maximum/README.md) — 单调队列滑窗
- 0295\. Find Median from Data Stream (待补) — **双堆**维护中位数 (流式设计进阶)
- 0480\. Sliding Window Median (待补) — 双堆 + 延迟删除
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — 矩阵 + 堆
- 0692\. Top K Frequent Words (待补) — 频次 + 字典序

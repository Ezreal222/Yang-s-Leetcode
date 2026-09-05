# 0023. Merge k Sorted Lists / 合并 K 个升序链表

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Linked List, Heap, Divide and Conquer, Merge Sort · 链表, 堆, 分治, 归并
    - **Link**: [LeetCode](https://leetcode.com/problems/merge-k-sorted-lists/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Combine k sorted lists into one, both in O(N log k)**. Two shapes: **pairwise merge in log k rounds** (reuse `mergeTwoLists`), or **min-heap of the k heads** (pop smallest, push its `.next`). Naive "merge into accumulator" is O(N·k) — reject.
>
> **中文**: **k 条有序链表合成一条, 两法都是 O(N log k)**. **两两归并 log k 轮** (复用 [0021](../0021-merge-two-sorted-lists/README.md)) 或 **k 个头节点入小根堆**, 弹最小 push 它的 next. 顺序拼 O(N·k), 淘汰.
>
> *Template / 模版*: **k-way merge** — 递归合并树 or 堆调度; 遇"多路归并" 二选一.

## Problem

**EN**: 给 k 条已排序的链表, 合成一条整体升序的链表.

**中文**: 合并 k 条有序链表.

## Key Insights

1. **🔑 灵魂: 为啥 O(N·k) 顺序合并是错的 / Why naive is bad**

    最直觉写法: 把 lists[0] 当 accumulator, 依次跟 lists[1], lists[2], ..., lists[k-1] 合并. 分析:

    - 每次 `mergeTwoLists(acc, lists[i])` 是 O(len(acc) + len(lists[i])).
    - 到第 i 轮, `acc` 已经吃了前 i 条链表的所有节点 → 长度 ~ i·(N/k).
    - 总 = Σ (i·N/k + N/k) = O(N·k / 2) = **O(N·k)**.

    最坏 N = 10⁴, k = 10⁴ → 10⁸ ops, TLE. → **必须做到 O(N log k)**.

    > **"顺序处理是 O(N·k), 分治/堆是 O(N log k)"** — 多路归并的经典 tradeoff.

2. **🔑 灵魂: 两两归并 O(N log k) — 每轮减半 / Pairwise merge halves rounds**

    Yang v1: 每轮把 `lists[i]` 和 `lists[i + k]` 归并, 一半装回原位.

    ```cpp
    while (n > 1) {
        int k = (n + 1) / 2;                          // ⌈n/2⌉, 处理奇数
        for (int i = 0; i < n / 2; i++) {             // 只走前半, 因为后半是"对方"
            lists[i] = mergeTwoLists(lists[i], lists[i + k]);
        }
        n = k;                                        // 下一轮长度
    }
    return lists[0];
    ```

    **正确性**: 每轮之后, 前 `⌈n/2⌉` 位置是各对的合并结果, 数量减半. log k 轮结束, 只剩 lists[0].

    **复杂度**: 每一轮的**总工作量 = N** (所有节点各被访问一次); log k 轮 → **O(N log k)**.

    > **v1 极简 in-place**: 直接在原 `lists` 数组里合并, 无额外容器. `n = (n + 1) / 2` 是⌈n/2⌉写法.

3. **🔑 灵魂: k-heap of heads O(N log k) — 池子每次拿最小 / Heap version**

    Yang v2: 建**小根堆**存 k 条链表的当前**头节点**. 每次:

    1. Pop 最小的 node.
    2. 挂到结果末尾.
    3. 若 `node->next` 非空, push 进堆.

    **复杂度**: 堆大小始终 ≤ k, 每次 O(log k), 共 N 次 → **O(N log k)**.

    ```cpp
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };   // 大于 → 小根
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
    ```

    - **`>` 变成小根堆** — C++ `priority_queue` 默认大根, comparator 返 `a > b` 反转.
    - **lambda + `decltype(cmp)`** — 面试常见 C++17 语法, 必背.
    - **`tail->next = nullptr`** — 最后手动切尾, 防止误挂到旧节点.

4. **🔑 v1 vs v2 tradeoff**

    | 维度 | v1 两两归并 | v2 堆 |
    |---|---|---|
    | 时间 | O(N log k) | O(N log k) |
    | 空间 | **O(1)** (in-place, mergeTwoLists 迭代版) | O(k) 堆 |
    | 代码量 | 短, 复用 [0021](../0021-merge-two-sorted-lists/README.md) | 中等, 需 comparator |
    | 常数 | **稍好** — 无堆维护 | 稍差 |
    | 递归深度 | mergeTwoLists 递归可爆栈 (N > 10⁴) | 无递归 |

    > **面试推 v1**: 简洁 + 复用 0021, 但**mergeTwoLists 要用迭代版避免爆栈**. v2 更通用 (多路归并模板).

5. **🔑 递归 mergeTwoLists 的栈风险 / Recursive merge stack risk**

    Yang v1 的 `mergeTwoLists` 是**递归版**. 若两条链共 20000 节点, 递归深度可达 20000, **爆栈**.

    → 生产/长链表: 改**迭代版**:

    ```cpp
    ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
        ListNode dummy(0);
        ListNode* tail = &dummy;
        while (a && b) {
            if (a->val < b->val) { tail->next = a; a = a->next; }
            else                 { tail->next = b; b = b->next; }
            tail = tail->next;
        }
        tail->next = a ? a : b;
        return dummy.next;
    }
    ```

    LC 判 20000 深度不爆 (默认 stack 通常 1MB, 递归帧 ~50-100 bytes), 但**面试主动提"若数据大, 改迭代"**.

6. **🔑 `n = (n + 1) / 2` 处理奇数 / Ceiling division for odd n**

    若 `n = 5`: 归并 `(0,3) (1,4)`, 第 2 位没配对. `k = (5+1)/2 = 3`, 循环 `i < n/2 = 2` 恰好归并 2 对, 第 2 位保留. 下轮 n = 3.

    → **奇数中间元素保留在 `lists[k-1]` 位置**, 下轮参与新一轮配对.

    > **`(n + 1) / 2` = `⌈n/2⌉`**, 处理奇偶通吃. 面试常用.

7. **🔑 边界: 空链 / Empty lists**

    - `lists` 数组为空 → 返 `nullptr`.
    - `lists[i]` 单条为空 → v2 里**不入堆** (`if (l) pq.push(l)`). v1 里 `mergeTwoLists(nullptr, x)` 自动返 x.

    > **易错点 top 1**: heap 版忘过滤 `nullptr` 头 → 崩溃 (访问 `nullptr->val`).

8. **🔑 复杂度总结 / Complexity summary**

    | 版本 | Time | Space |
    |---|---|---|
    | **顺序合并 (naive)** | O(N·k) | O(1) |
    | **两两归并** | **O(N log k)** | O(1) / 递归栈 O(N) |
    | **堆** | **O(N log k)** | O(k) |

## Solution

=== "C++ (v1: 两两归并, 推荐)"
    ```cpp
    class Solution {
    public:
        ListNode* mergeKLists(vector<ListNode*>& lists) {
            if (lists.empty()) return nullptr;
            int n = lists.size();
            while (n > 1) {
                int k = (n + 1) / 2;                              // ⌈n/2⌉ 处理奇数
                for (int i = 0; i < n / 2; i++) {
                    lists[i] = mergeTwoLists(lists[i], lists[i + k]);
                }
                n = k;                                            // 下一轮长度
            }
            return lists[0];
        }

        // 复用 0021, 递归版; 长链表建议改迭代
        ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
            if (!list1) return list2;
            if (!list2) return list1;
            if (list1->val < list2->val) {
                list1->next = mergeTwoLists(list1->next, list2);
                return list1;
            } else {
                list2->next = mergeTwoLists(list1, list2->next);
                return list2;
            }
        }
    };
    ```

=== "C++ (v2: 小根堆)"
    ```cpp
    class Solution {
    public:
        ListNode* mergeKLists(vector<ListNode*>& lists) {
            // lambda + decltype: C++17 声明堆的 comparator 惯用招
            // 返 a->val > b->val 让默认"大根" 反转成小根
            auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
            priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

            for (auto l : lists) if (l) pq.push(l);               // 空指针不入堆!

            ListNode dummy(0);
            ListNode* tail = &dummy;

            while (!pq.empty()) {
                ListNode* node = pq.top(); pq.pop();
                tail->next = node;
                tail = node;
                if (node->next) pq.push(node->next);              // 跟进下一位
            }
            tail->next = nullptr;                                 // 手动切尾防悬挂
            return dummy.next;
        }
    };
    ```

=== "Python (heapq 版)"
    ```python
    import heapq

    class Solution:
        def mergeKLists(self, lists: list['ListNode | None']) -> 'ListNode | None':
            # heapq 默认小根堆, 直接用.
            # 但 ListNode 无 __lt__ 定义, tie 时会 TypeError → 用 (val, idx, node) 三元组
            # idx 保证任意两 tuple 可比 (int 天生可比), 避开 node 比较
            heap = []
            for i, node in enumerate(lists):
                if node:
                    heapq.heappush(heap, (node.val, i, node))

            dummy = ListNode(0)
            tail = dummy
            # 用 counter 生成新 idx (每次 push 递增), 保证唯一
            counter = len(lists)
            while heap:
                val, _, node = heapq.heappop(heap)
                tail.next = node
                tail = node
                if node.next:
                    heapq.heappush(heap, (node.next.val, counter, node.next))
                    counter += 1
            tail.next = None
            return dummy.next
    ```

=== "JavaScript (两两归并版)"
    ```javascript
    // JS 无原生 heap. 面试可 (a) 手写小堆, (b) 用两两归并 (推荐, 短)
    // 这里选 (b) 复用 0021 的合并逻辑

    var mergeKLists = function(lists) {
        if (lists.length === 0) return null;

        // 迭代版 mergeTwoLists 避免爆栈; 相当于 C++ 迭代版
        const mergeTwo = (a, b) => {
            const dummy = new ListNode(0);
            let tail = dummy;
            while (a && b) {
                if (a.val < b.val) { tail.next = a; a = a.next; }
                else               { tail.next = b; b = b.next; }
                tail = tail.next;
            }
            tail.next = a ?? b;                                  // ?? 是 null-coalescing
            return dummy.next;
        };

        // 两两归并, 每轮减半, 共 log k 轮
        let n = lists.length;
        while (n > 1) {
            const k = Math.ceil(n / 2);                          // ⌈n/2⌉
            for (let i = 0; i < Math.floor(n / 2); i++) {
                lists[i] = mergeTwo(lists[i], lists[i + k]);
            }
            n = k;
        }
        return lists[0];
    };
    ```

## Complexity

| 版本 | Time | Space |
|---|---|---|
| **两两归并** | **O(N log k)** | O(1) |
| **小根堆** | **O(N log k)** | O(k) |

其中 N = 所有节点总数, k = 链表数.

## 相关题目

- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — 母题, 两两归并的复用零件
- 0148\. Sort List (待补) — 链表归并排序, 分治招同源
- [0347. Top K Frequent Elements](../../06-stack-queue/0347-top-k-frequent-elements/README.md) — 堆的另一用法
- [0215. Kth Largest Element in an Array](../../06-stack-queue/0215-kth-largest-element-in-an-array/README.md) — heap select
- [0692. Top K Frequent Words](../../06-stack-queue/0692-top-k-frequent-words/README.md) — heap + tie-break
- [0703. Kth Largest Element in a Stream](../../06-stack-queue/0703-kth-largest-element-in-a-stream/README.md) — 流式 min-heap
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — 多路归并 + 堆
- 0632\. Smallest Range Covering Elements from K Lists (待补) — 多路 + 堆 + 滑窗
- 0264\. Ugly Number II (待补) — k-way merge 思想

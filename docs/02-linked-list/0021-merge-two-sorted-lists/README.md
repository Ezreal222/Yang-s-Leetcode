# 0021. Merge Two Sorted Lists / 合并两个有序链表

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Linked List, Dummy Head, Recursion, Two Pointers · 链表, 虚拟头节点, 递归, 双指针
    - **Link**: [LeetCode](https://leetcode.com/problems/merge-two-sorted-lists/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Merge two sorted lists in-place** → **v1 iterative + dummy**: pick smaller head each step, splice, tail-attach leftover. **v2 recursive**: pick smaller as head, recurse on rest, `next = recurse(...)`.
>
> **中文**: **原地合并两有序链** → **v1 迭代 + dummy**: 每步取小的接过来, 剩余整段直接拼. **v2 递归**: 取小作头, 递归接剩余.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给两个升序链表, **合并**成一个升序链表并返回头节点. **复用**原节点 (不建新节点).

**中文**: 合并两个升序链表.

## Key Insights

1. **🔑 又见 dummy: 免"第一个节点特判" / Dummy head again**

    没 dummy 时, 处理**第一个节点** 要特殊 (谁小谁作 head, 之后再统一处理). **加 dummy 后**, 起点 `curr = &dummy`, **所有节点** 都是"接到 curr 后面" — **统一逻辑**.

    > **头可能变 → 加 dummy**. [0203](../0203-remove-linked-list-elements/README.md) / [0707](../0707-design-linked-list/README.md) / [0024](../0024-swap-nodes-in-pairs/README.md) / [0019](../0019-remove-nth-node-from-end-of-list/README.md) 全家福.

2. **🔑 v1 迭代: 双指针 + curr 跟随 / Iterative: two pointers + running curr**

    ```cpp
    while (list1 && list2) {
        if (list1->val <= list2->val) {
            curr->next = list1;
            list1 = list1->next;
        } else {
            curr->next = list2;
            list2 = list2->next;
        }
        curr = curr->next;
    }
    ```

    - **每步**: 比 list1 / list2 头, 选**小**的接到 curr 后, 该链前进一位, curr 前进一位.
    - **`<=` 稳定**: 相等时优先 list1 → 保持原序.

3. **🔑 收尾拼接: `curr->next = list1 ? list1 : list2` / Tail attach the remainder**

    循环出来后, **至少一条空**. 剩下的**整段已排序**, **直接接** 到 curr 后, 不用一个个再遍历.

    - `list1 ? list1 : list2` — 用**三目**兼顾"哪个非空"; 若两个都空 (罕见), 接 nullptr 也对.

    > **"剩余已排好 → 一挂了之"** 是 merge 的高效收尾. 每节点访问 1 次.

4. **🔑 复用节点, 不建新 / Reuse nodes, no allocation**

    题目允许原地修改 next 指针 — **不需要 `new ListNode`**. 更省内存, 也符合"合并 = 重排指针" 的直觉.

5. **🔑 v2 递归: 简洁 4 行 / Recursive: 4 lines**

    ```cpp
    if (!list2) return list1;
    if (!list1) return list2;
    if (list1->val <= list2->val) {
        list1->next = mergeTwoLists(list1->next, list2);
        return list1;
    } else {
        list2->next = mergeTwoLists(list1, list2->next);
        return list2;
    }
    ```

    - **base**: 一空返另一.
    - **归纳**: 选小的作 head, **递归合并剩余** 作它的 next, 返 head.
    - **信任子问题**: 递归返"剩余合并结果的头".

    > **递归解链表的美感** — "每一层只做一件事: 选头 + 委托剩余". 跟 [0206 递归反转](../0206-reverse-linked-list/README.md) / [0024 递归交换](../0024-swap-nodes-in-pairs/README.md) 同族.

6. **🔑 v1 vs v2 对比 / v1 vs v2**

    | | **v1 迭代** | **v2 递归** |
    |---|---|---|
    | Time | O(m + n) | O(m + n) |
    | Space | **O(1)** | O(m + n) (调用栈) |
    | 代码 | 长 (~10 行) | **短 (~5 行)** |
    | 溢出风险 | 无 | 长链 stack overflow |
    | 面试推荐 | **首选** | 展示递归 |

    > **工程首选迭代** (无 stack overflow 风险). **递归**是"我也会递归" 的展示.

7. **🔑 复杂度 O(m + n) 时间 / Linear time**

    每节点被访问 1 次 (选或递归展开). 迭代 O(1) 空间, 递归 O(m + n) 栈.

8. **🔑 应用: 归并排序链表基础 / Foundation for merge-sort linked list**

    0148 Sort List 用**归并排序**, 每次分两半, 用**本题** merge 合. 是链表排序的 canonical 解.

## Solution

=== "C++"

    **v1: 迭代 + dummy (首选)**

    ```cpp
    class Solution {
    public:
        ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
            ListNode dummy(0);                                       // 栈上 dummy
            ListNode* curr = &dummy;
            while (list1 && list2) {
                if (list1->val <= list2->val) {                      // <= 稳定
                    curr->next = list1;
                    list1 = list1->next;
                } else {
                    curr->next = list2;
                    list2 = list2->next;
                }
                curr = curr->next;
            }
            curr->next = list1 ? list1 : list2;                      // 挂剩余
            return dummy.next;
        }
    };
    ```

    **v2: 递归 (简洁)**

    ```cpp
    class Solution {
    public:
        ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
            if (!list2) return list1;
            if (!list1) return list2;
            if (list1->val <= list2->val) {
                list1->next = mergeTwoLists(list1->next, list2);
                return list1;
            } else {
                list2->next = mergeTwoLists(list1, list2->next);
                return list2;
            }
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1 迭代
        def mergeTwoLists(self, list1, list2):
            dummy = ListNode(0)
            cur = dummy
            while list1 and list2:
                if list1.val <= list2.val:
                    cur.next = list1
                    list1 = list1.next
                else:
                    cur.next = list2
                    list2 = list2.next
                cur = cur.next
            # `or` 短路: list1 非空返 list1, 否则返 list2 (可能 None)
            cur.next = list1 or list2
            return dummy.next

        # v2 递归 — 4 行
        def mergeTwoLists_rec(self, list1, list2):
            if not list1: return list2
            if not list2: return list1
            if list1.val <= list2.val:
                list1.next = self.mergeTwoLists_rec(list1.next, list2)
                return list1
            list2.next = self.mergeTwoLists_rec(list1, list2.next)
            return list2
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} list1
     * @param {ListNode} list2
     * @return {ListNode}
     */
    var mergeTwoLists = function(list1, list2) {
        // v1 迭代. JS 用 { val, next } 对象当 dummy
        const dummy = { val: 0, next: null };
        let cur = dummy;
        while (list1 && list2) {
            if (list1.val <= list2.val) {
                cur.next = list1;
                list1 = list1.next;
            } else {
                cur.next = list2;
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 || list2;          // JS 短路
        return dummy.next;
    };
    ```

## Complexity

- **Time**: O(m + n) — 每节点访问 1 次.
- **Space**: **O(1) 迭代** / O(m + n) 递归 (调用栈).

## 相关题目

- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题
- [0024. Swap Nodes in Pairs](../0024-swap-nodes-in-pairs/README.md) — dummy + 三指针
- [0025. Reverse Nodes in k-Group](../0025-reverse-nodes-in-k-group/README.md) — 半开区间 + helper 返值
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap
- [0138. Copy List with Random Pointer](../0138-copy-list-with-random-pointer/README.md) — 复杂链表深拷贝
- 0023\. Merge k Sorted Lists (待补, Hard) — k 路合并, 优先队列 / 分治
- 0148\. Sort List (待补) — 链表归并排序, 用本题作 merge
- 0086\. Partition List (待补) — dummy × 2 分组
- 0328\. Odd Even Linked List (待补) — 奇偶分离
- 0088\. Merge Sorted Array (待补) — 数组版, 从后往前合

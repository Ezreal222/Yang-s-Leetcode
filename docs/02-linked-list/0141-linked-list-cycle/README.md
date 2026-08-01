# 0141. Linked List Cycle / 环形链表

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Linked List, Two Pointers, Floyd's Cycle Detection · 链表, 双指针, Floyd 判环
    - **Link**: [LeetCode](https://leetcode.com/problems/linked-list-cycle/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Detect cycle in linked list, O(1) space** → **Floyd tortoise/hare Phase 1**: fast (2×) / slow (1×); if they meet ⇒ cycle; if fast hits null ⇒ no cycle.
>
> **中文**: **判链表有环, O(1) 空间** → **Floyd 龟兔阶段 1**: fast (2×) / slow (1×); 相遇即有环, fast 到 null 即无环.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单链表, 判是否**有环**. **O(1) 空间**.

**中文**: 判环. 空间 O(1).

## Key Insights

1. **🔑 Floyd 阶段 1 — 只判"有无环" / Phase 1 only: cycle existence**

    ```cpp
    while (fast && fast->next) {
        fast = fast->next->next;             // 2 步
        slow = slow->next;                    // 1 步
        if (fast == slow) return true;        // 相遇 = 有环
    }
    return false;                             // fast 到 null = 无环
    ```

    - **相遇必发生 (若有环)**: fast 比 slow 每步靠近 1, 环长有限 → 追上不超过 1 圈.
    - **无环时 fast 到 null**: 循环退出返 false.

    > **本题只要"有没有" — 不用做 Phase 2 (找入口)**. Phase 2 见 [0142 Linked List Cycle II](../0142-linked-list-cycle-ii/README.md).

2. **🔑 循环条件 `fast && fast->next` / Guarded march**

    fast 每步走两下, 必须**都非 null**. **顺序**: `fast && fast->next` 短路防 null deref (若 `fast == null` 就不算第二个).

    > **短路 + 顺序** — 二分 / 快慢指针的通用防御. 反了会 crash.

3. **🔑 备选: hash set / Alternative**

    ```cpp
    unordered_set<ListNode*> seen;
    while (head) {
        if (seen.count(head)) return true;
        seen.insert(head);
        head = head->next;
    }
    return false;
    ```

    - O(n) 时间, **O(n) 空间**.
    - **面试要 O(1) 空间** → 上 Floyd.

4. **🔑 跟 [0142 Cycle II](../0142-linked-list-cycle-ii/README.md) 的关系: 阶段 1 vs 完整 / vs 0142**

    | | **0141 (本题)** | 0142 |
    |---|---|---|
    | 问 | 有没有环 | 环入口在哪 |
    | 阶段 | Phase 1 only | Phase 1 + Phase 2 |
    | 数学 | 无需 | `a = c + (n-1)·环长` 推导 |
    | 输出 | bool | 节点指针 |

    > 学一得二 — 0141 熟了 0142 就加一段 Phase 2.

5. **🔑 快慢指针家族回顾 / Fast-slow family**

    | 变种 | 差距 | 用途 | 例题 |
    |---|---|---|---|
    | 同步差 n | 固定 | 倒数第 n | [0019](../0019-remove-nth-node-from-end-of-list/README.md) |
    | 差速 2×/1× | 变量 | 找中点 / **判环** | 0876 / **本题** |
    | Floyd 追赶 | 变 | 找环入口 | [0142](../0142-linked-list-cycle-ii/README.md) |

6. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear time**

    - 无环: fast 走 n/2 步到 null.
    - 有环: 追上不超过环长 (< n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool hasCycle(ListNode* head) {
            ListNode* slow = head;
            ListNode* fast = head;
            while (fast && fast->next) {
                fast = fast->next->next;
                slow = slow->next;
                if (fast == slow) return true;
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def hasCycle(self, head) -> bool:
            slow = fast = head
            while fast and fast.next:
                fast = fast.next.next
                slow = slow.next
                # `is` 比 `==` 更贴指针语义 (等价性 vs 值相等)
                if slow is fast:
                    return True
            return False
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {boolean}
     */
    var hasCycle = function(head) {
        let slow = head, fast = head;
        while (fast && fast.next) {
            fast = fast.next.next;
            slow = slow.next;
            if (slow === fast) return true;    // === 严格相等 (引用比较)
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(n) — 无环 n/2 步到底, 有环 ≤ n/2 + 环长.
- **Space**: **O(1)** — 两指针.

## 相关题目

- [0142. Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) — 进阶找环入口, Floyd 数学
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap
- [0202. Happy Number](../../03-hash-table/0202-happy-number/README.md) — Floyd 用在数字序列
- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — 双指针合并
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题
- 0287\. Find the Duplicate Number (待补) — Floyd 用在数组
- 0876\. Middle of the Linked List (待补) — 2×/1× 找中点
- 0457\. Circular Array Loop (待补) — 环检测变体

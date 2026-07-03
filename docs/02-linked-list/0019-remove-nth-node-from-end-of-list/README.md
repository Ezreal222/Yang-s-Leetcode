# 0019. Remove Nth Node From End of List / 删除链表的倒数第 N 个节点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Two Pointers, Fast-Slow, Dummy Head · 链表, 双指针, 快慢指针, 虚拟头节点
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **One-pass remove nth from end** → **dummy + fast/slow with gap of `n`**: advance fast by n first, then both march until `fast->next` is null → slow lands on **predecessor** of target.
>
> **中文**: **一遍扫删倒数第 n** → **dummy + 快慢差 n**: fast 先走 n 步, 再一起走到 fast 停在尾 → slow 恰好是**被删节点的前驱**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单链表, 删**倒数第 n 个** 节点, 返回新头. 尝试**一次遍历**.

**中文**: 一遍扫删倒数第 n.

## Key Insights

1. **🔑 一遍扫的关键: 用 gap 编码"倒数位置" / One-pass trick: encode "from-end" with a gap**

    传统**两遍**思路: 先扫得长度 L, 再扫走 `L - n` 步定位. 直观但**扫两遍**.

    **快慢指针**: 让 fast 比 slow 领先 **n 步**, 然后**同步推进**. 当 fast 到尾时, slow 距尾**恰好还有 n 步** → slow 是**倒数第 (n+1)** = 被删节点的**前驱**.

    > 用**相对距离** 代替**绝对长度** — 双指针题的核心思维.

2. **🔑 dummy 必不可少 (删头 corner case) / Dummy is mandatory**

    若 `n == length` (删头节点), slow 需要"头的前驱" — **没 dummy 就没这东西**. 加 dummy 后, slow 起点 = dummy, 天然覆盖:

    - 删头: slow 一直是 dummy, 删的是 `dummy->next` = head.
    - 删中间 / 尾: slow 走到 target 前驱.

    > **凡是可能改头的链表操作 → 加 dummy**. [0203](../0203-remove-linked-list-elements/README.md) / [0707](../0707-design-linked-list/README.md) / [0024](../0024-swap-nodes-in-pairs/README.md) 反复验证.

3. **🔑 fast 走 n 步不是 n+1 步 / Fast advances by n, not n+1**

    循环条件 `while (fast->next)` (不是 `while (fast)`) — fast 停在**最后一个真节点**, 不是 null.

    分析 gap:

    ```
    dummy → A → B → C → D → null    (length 4, n = 2, 目标 = C)
              slow (最终)
                        fast (最终)
    ```

    - fast 起 dummy, 走 n=2 步 → fast 到 B.
    - 一起走: slow=dummy→A, fast=B→C.
    - 一起走: slow=A→B, fast=C→D.
    - `fast->next == null` (D 是尾) → 停. slow = B = **C 的前驱** ✅.

    > **循环用 `fast->next` 不是 `fast`** 是本题精髓. 若用 `fast`, fast 会多走一步到 null, slow 会指到 C 本身 (不是 C 的前驱), 就删不了.

4. **🔑 可视化: 拉绳子的两个人 / Two runners with a rope**

    > **前面的人 (fast) 拉着长 n 的绳走**, 到墙就停. **后面的人 (slow) 距墙 = 绳长 n**. 想删"倒数第 n" → 后面的人手里刚好是"该删节点的前驱".

    这个比喻能帮记 gap = n 而不是 n+1 / n-1.

5. **🔑 删除的三步 (跟 [0203](../0203-remove-linked-list-elements/README.md) 同款) / Delete in 3 lines (same as 0203)**

    ```cpp
    ListNode* toDel = slow->next;
    slow->next = toDel->next;
    delete toDel;                          // 工程素养
    ```

    - 暂存要删的.
    - 前驱跳过它.
    - `delete` 释放 (LC 不检查, 但工程要写).

6. **🔑 fast/slow 是**双指针**的一族 / Fast-slow is a two-pointer family**

    双指针大类里, **快慢** 又分几种:

    | 变种 | 差距 | 用途 | 例题 |
    |---|---|---|---|
    | 同步差 n | 固定 n | 倒数第 n | **本题** |
    | 差速 (fast 2×, slow 1×) | 变量 | 找中点 / 判环 | 0876 / 0141 |
    | 追赶 (fast 从中间起) | 变 | Floyd 判环入口 | 0142 |

    > 同一"快慢" 家族三种玩法, 记住**差距怎么用**.

7. **复杂度 O(L) 时间, O(1) 空间 / Linear, constant**

    L = 链长. 一遍扫 (fast + slow 各扫一次), 两指针常数空间.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* removeNthFromEnd(ListNode* head, int n) {
            ListNode dummy(0);
            dummy.next = head;
            ListNode* slow = &dummy;
            ListNode* fast = &dummy;

            for (int i = 0; i < n; i++) fast = fast->next;           // fast 先走 n 步

            while (fast->next) {                                     // ← 用 fast->next 不是 fast
                slow = slow->next;
                fast = fast->next;
            }
            // 现在 slow 是被删节点的前驱

            ListNode* toDel = slow->next;
            slow->next = toDel->next;
            delete toDel;

            return dummy.next;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def removeNthFromEnd(self, head, n: int):
            # Python: GC 兜底, 不用 delete. 用 ListNode 实例当 dummy
            dummy = ListNode(0, head)
            slow = fast = dummy
            # 双赋值 slow = fast = dummy 让两个指针指同一节点, 跟 C++ 两行等价
            for _ in range(n):
                fast = fast.next        # fast 先走 n 步

            while fast.next:            # 用 fast.next, 停在最后一个真节点
                slow = slow.next
                fast = fast.next

            # 前驱跳过 slow.next, GC 回收被删节点
            slow.next = slow.next.next
            return dummy.next
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @param {number} n
     * @return {ListNode}
     */
    var removeNthFromEnd = function(head, n) {
        const dummy = { val: 0, next: head };
        let slow = dummy, fast = dummy;
        // let a = b = c 在 JS 里有作用域坑 (b, c 会成 global) — 分开写更安全
        for (let i = 0; i < n; i++) fast = fast.next;
        while (fast.next) {
            slow = slow.next;
            fast = fast.next;
        }
        slow.next = slow.next.next;     // GC 自动清理
        return dummy.next;
    };
    ```

## Complexity

- **Time**: O(L) — 一遍扫, L = 链长.
- **Space**: O(1) — dummy + 两指针.

## 相关题目

- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题, 删除操作模板
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 链表反转母题
- [0024. Swap Nodes in Pairs](../0024-swap-nodes-in-pairs/README.md) — dummy + 三指针
- [0025. Reverse Nodes in k-Group](../0025-reverse-nodes-in-k-group/README.md) — 快慢 gap 定位 + 反转
- 0876\. Middle of the Linked List (待补) — fast 2× slow, 找中点
- 0141\. Linked List Cycle (待补) — fast/slow 判环
- [0142. Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) — Floyd 找环入口
- 0061\. Rotate List (待补) — 类似"倒数第 k" 定位

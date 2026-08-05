# 0143. Reorder List / 重排链表

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Two Pointers, Reverse, Merge · 链表, 双指针, 反转, 合并
    - **Link**: [LeetCode](https://leetcode.com/problems/reorder-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Reorder L₀→L₁→…→Lₙ₋₁ into L₀→Lₙ₋₁→L₁→Lₙ₋₂→…** → **three-step combo**: (1) fast/slow find **middle**; (2) **reverse** second half; (3) **weave-merge** first + reversed-second alternately.
>
> **中文**: **L₀→L₁→…→Lₙ₋₁ 重排成 L₀→Lₙ₋₁→L₁→Lₙ₋₂→…** → **三合一**: (1) 快慢**找中点**; (2) **反转** 后半段; (3) **交替 merge** 前半 + 反转后半.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给单链表 `head`. **原地** 重排为 `L₀ → Lₙ₋₁ → L₁ → Lₙ₋₂ → L₂ → …`.

- 例: `1→2→3→4→5` → `1→5→2→4→3`.

**中文**: 首尾交替重排链表, 原地.

## Key Insights

1. **🔑 灵魂: 三个基本操作拼装 / Three-step combo**

    ```
    Step 1: 找中点 → 前半 [head..slow], 后半 (slow..null]
    Step 2: 反转后半 → 反转后头
    Step 3: 交替 merge 前半 + 反转后半
    ```

    **每一步**都是链表基本操作的组合. 这是**链表 lego 拼装** 思维.

    > **链表 Hard 题很多是三 / 四个基本操作组合**. 学好 0876 中点 + [0206 反转](../0206-reverse-linked-list/README.md) + [0021 merge](../0021-merge-two-sorted-lists/README.md) 三个母题, 组合题就顺.

2. **🔑 Step 1: 快慢找中点 (中偏左) / Fast-slow: middle (left-biased for even)**

    ```cpp
    ListNode *slow = head, *fast = head;
    while (fast->next && fast->next->next) {         // 关键条件
        fast = fast->next->next;
        slow = slow->next;
    }
    ```

    - **`fast->next && fast->next->next`** — 让 slow 停在**前半的末尾** (中偏左).
    - **n = 4**: `1→2→3→4`, slow 停在 index 1 (值 2). 前半 [1,2], 后半 [3,4].
    - **n = 5**: `1→2→3→4→5`, slow 停在 index 2 (值 3). 前半 [1,2,3], 后半 [4,5].

    > **中偏左的选择**: 让**前半 ≥ 后半** — merge 时第一段可以领先, 收尾自然. 若用**中偏右** (`fast && fast->next`), 后半 ≥ 前半, 收尾多一步.

3. **🔑 Step 1.5: 断开前后 / Cut**

    ```cpp
    ListNode* second = slow->next;
    slow->next = nullptr;                            // 关键断开!
    ```

    **必须断开** — 否则 Step 3 merge 时前半末尾还指向后半头, 会**死循环** (交叉后回来).

    > **切链子**是链表操作的基本动作. 断了之后**两段独立**, 好处理.

4. **🔑 Step 2: 反转后半 / Reverse second half**

    ```cpp
    ListNode* prev = nullptr;
    while (second) {
        ListNode* next = second->next;
        second->next = prev;
        prev = second;
        second = next;
    }
    second = prev;                                    // 反转后的新头
    ```

    直接搬 [0206 迭代反转](../0206-reverse-linked-list/README.md) 模板.

    > **模板复用**: 反转的四步"备→翻→推 prev→推 curr" 一辈子的工具.

5. **🔑 Step 3: 交替 merge / Weave-merge**

    ```cpp
    while (second) {
        ListNode* t1 = first->next;
        ListNode* t2 = second->next;
        first->next = second;
        second->next = t1;
        first = t1;
        second = t2;
    }
    ```

    - **每步**接一个 first + 一个 second, 交替.
    - **备份 `t1, t2`**: 因为 `first->next` 和 `second->next` 都会被改, **必须先存**.
    - **循环 `while (second)`**: 前半 ≥ 后半 (中偏左保证) → second 先走完, 剩余 first 自然挂着 (前半末尾).

6. **🔑 为啥用 `while (second)` 而非 `while (first)`? / Why loop on second**

    因为**前半 ≥ 后半**: 若循环 first, 后半会先耗完但 first 还没到 null, 得内部判. **循环 second 更干净** — 后半走完 = 结束.

    最后**前半末尾节点** (若前半多 1 个) 自然保留原 next = nullptr (因 Step 1.5 已断开).

7. **🔑 跟 [0234 Palindrome Linked List] 是同款套路 / vs 0234**

    | | **0143 (本题)** | 0234 Palindrome |
    |---|---|---|
    | 步骤 1 | 快慢找中 | **一样** |
    | 步骤 2 | 反转后半 | **一样** |
    | 步骤 3 | **交替 merge** | **逐节点比较** |

    > **同族**"三合一" 链表题. 学一得二.

8. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear time, constant space**

    - Step 1 O(n/2), Step 2 O(n/2), Step 3 O(n/2). 总 O(n).
    - 只用常数指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        void reorderList(ListNode* head) {
            // Step 1: 找中点 (中偏左)
            ListNode *slow = head, *fast = head;
            while (fast->next && fast->next->next) {
                fast = fast->next->next;
                slow = slow->next;
            }

            // Step 1.5: 切断
            ListNode* second = slow->next;
            slow->next = nullptr;

            // Step 2: 反转后半
            ListNode* prev = nullptr;
            while (second) {
                ListNode* next = second->next;
                second->next = prev;
                prev = second;
                second = next;
            }
            second = prev;

            // Step 3: 交替 merge
            ListNode* first = head;
            while (second) {
                ListNode* t1 = first->next;
                ListNode* t2 = second->next;
                first->next = second;
                second->next = t1;
                first = t1;
                second = t2;
            }
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def reorderList(self, head) -> None:
            # Step 1: 找中点 (中偏左)
            slow = fast = head
            while fast.next and fast.next.next:
                fast = fast.next.next
                slow = slow.next

            # Step 1.5: 切断
            second = slow.next
            slow.next = None

            # Step 2: 反转后半 — 元组解包一行搞定四步
            prev = None
            while second:
                second.next, prev, second = prev, second, second.next
            second = prev

            # Step 3: 交替 merge
            first = head
            while second:
                t1, t2 = first.next, second.next
                first.next = second
                second.next = t1
                first, second = t1, t2
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {void} Do not return anything, modify head in-place instead.
     */
    var reorderList = function(head) {
        // Step 1: 找中点
        let slow = head, fast = head;
        while (fast.next && fast.next.next) {
            fast = fast.next.next;
            slow = slow.next;
        }
        // Step 1.5: 切断
        let second = slow.next;
        slow.next = null;
        // Step 2: 反转后半
        let prev = null;
        while (second) {
            const next = second.next;
            second.next = prev;
            prev = second;
            second = next;
        }
        second = prev;
        // Step 3: 交替 merge
        let first = head;
        while (second) {
            const t1 = first.next;
            const t2 = second.next;
            first.next = second;
            second.next = t1;
            first = t1;
            second = t2;
        }
    };
    ```

## Complexity

- **Time**: O(n) — 三步各 O(n/2).
- **Space**: O(1) — 常数指针.

## 相关题目

- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题 (Step 2 复用)
- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — merge 母题 (Step 3 变形)
- [0142. Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) — Floyd 判环入口
- [0141. Linked List Cycle](../0141-linked-list-cycle/README.md) — 快慢判环
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap
- [0138. Copy List with Random Pointer](../0138-copy-list-with-random-pointer/README.md) — 复杂链表深拷贝
- 0876\. Middle of the Linked List (待补) — Step 1 母题
- 0234\. Palindrome Linked List (待补) — **同款三合一**, 步骤 3 换比较
- 0061\. Rotate List (待补) — 部分旋转
- 0148\. Sort List (待补) — 归并排序链表

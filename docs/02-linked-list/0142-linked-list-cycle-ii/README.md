# 0142. Linked List Cycle II / 环形链表 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Two Pointers, Floyd's Algorithm, Cycle Detection · 链表, 双指针, Floyd 判环, 环检测
    - **Link**: [LeetCode](https://leetcode.com/problems/linked-list-cycle-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find cycle entry, O(1) space** → **Floyd's tortoise & hare**: Phase 1 fast(2×)/slow(1×) meet inside cycle; Phase 2 reset one to head, both step 1 → they meet **at cycle entry** (from math: `a = c + (n-1)·loop`).
>
> **中文**: **找环入口, O(1) 空间** → **Floyd 龟兔**: 阶段 1 fast 2×/slow 1× 环内相遇; 阶段 2 一个回 head, 两个每步 1 → 相遇在**入环点** (数学: `a = c + (n-1)·环长`).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单链表可能有环. 返回**环的入口**节点; 无环返 `nullptr`. 要求 **O(1) 空间**.

**中文**: 找环的入口节点, 无环返 null. 空间 O(1).

## Key Insights

1. **🔑 朴素 vs Floyd / Naive vs Floyd**

    | 思路 | Time | Space | 备注 |
    |---|---|---|---|
    | 哈希集: 走过就记, 见到重复 = 入口 | O(n) | **O(n)** | 直观 |
    | **Floyd 龟兔** (Yang) | O(n) | **O(1)** | 数学优雅 |

    > 面试要求 O(1) 空间 → 必须 Floyd. 若允许 O(n), hash set 更快写.

2. **🔑 Phase 1: 快慢 2× vs 1× 一定会在环内相遇 (若有环) / Phase 1: fast catches slow inside the cycle**

    ```cpp
    while (fast && fast->next) {
        slow = slow->next;              // 1 步
        fast = fast->next->next;        // 2 步
        if (slow == fast) break;        // 相遇
    }
    ```

    - **无环**: fast 会走到 null (或 fast->next 到 null) → 自然退出, `!fast || !fast->next` 判为无环.
    - **有环**: 一旦 slow 入环, fast 比 slow 每步快 1 → 环长内**必追上**. 追上时 slow == fast.

    > **为啥必追上?** 因为 fast 相对 slow 每步"接近" 1 (环内看), 环长有限 → 追上不超过 1 圈.

3. **🔑 Phase 2 数学: 灵魂推导 / The magical math**

    命名距离:

    - `a` = head → 入环点
    - `b` = 入环点 → 相遇点 (顺环方向)
    - `c` = 相遇点 → 入环点 (顺环方向)
    - 环长 = `b + c`

    ```
    head ─── a ───→ [入环点] ─── b ───→ [相遇点]
                          ↑                       │
                          └──────── c ────────────┘
    ```

    相遇时:

    - slow 走了 `a + b`
    - fast 走了 `a + b + n(b + c)` (n = fast 多绕的圈数 ≥ 1)
    - fast = 2 × slow: **`a + b + n(b + c) = 2(a + b)`**
    - 整理: **`a = (n − 1)(b + c) + c`**

    **含义**: **head 到入环点的距离 `a` = 相遇点走 `c` 步到入环点 (可能多绕 n-1 圈)**.

    → **head 和相遇点同时出发, 每步 1, 会同时到入环点**. ✅

    ```cpp
    ListNode* p = head;
    while (p != slow) {                     // slow 还站在相遇点
        p = p->next;
        slow = slow->next;
    }
    return p;                                // 相遇在入环点
    ```

    > **数学到代码的翻译只有 3 行**. 这题美在**几何 + 代数** 化成极简的编程步骤.

4. **🔑 循环条件 `fast && fast->next` / Guarded march for fast**

    fast 每步走两下, **必须**保证:

    - `fast != nullptr` (第一下不 crash)
    - `fast->next != nullptr` (第二下不 crash)

    顺序**必须** `fast && fast->next` — 短路防 null deref.

5. **🔑 判无环: 循环退出后再验一次 / Post-loop check for no-cycle**

    ```cpp
    if (!fast || !fast->next) return nullptr;
    ```

    循环用 `break` 出来 (相遇) → fast 一定非空 → 这行不走.

    循环**自然**退出 → 说明 fast 触底 → 无环. 这行拦截.

    > **别忘写这行**. 少了就返错 (slow 停在无环的某处而非环入口).

6. **🔑 三大快慢指针玩法 / Three flavors of fast-slow (回顾)**

    | 变种 | 差距 | 用途 | 例题 |
    |---|---|---|---|
    | 同步差 n | 固定 n | 倒数第 n | [0019](../0019-remove-nth-node-from-end-of-list/README.md) |
    | 差速 2× vs 1× | 变量 | 找中点 / 判环 (阶段 1) | 0876 / 0141 |
    | **Floyd 追赶** | 变 | **找环入口** | **本题** |

    > **同一种"快慢" 玩三种花样** — 差距怎么用是关键. Floyd 是最优雅的一种.

7. **复杂度 O(n) 时间, O(1) 空间 / Linear time, constant space**

    - Phase 1: fast 走最多 2n 步.
    - Phase 2: 走最多 n 步.
    - 空间: 3 个指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* detectCycle(ListNode* head) {
            ListNode *slow = head, *fast = head;

            // Phase 1: 找相遇点 (若有环)
            while (fast && fast->next) {
                slow = slow->next;
                fast = fast->next->next;
                if (slow == fast) break;
            }
            if (!fast || !fast->next) return nullptr;                // 循环自然退 → 无环

            // Phase 2: 一个回 head, 两个每步 1, 相遇即入环点
            ListNode* p = head;
            while (p != slow) {
                slow = slow->next;
                p = p->next;
            }
            return p;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def detectCycle(self, head):
            slow = fast = head
            # Phase 1
            while fast and fast.next:
                slow = slow.next
                fast = fast.next.next
                if slow is fast:            # `is` 比 `==` 更贴指针语义
                    break
            else:
                # Python for/while 独有的 else: 若循环自然结束 (没 break) 才执行
                # 这里等价 C++ 里的"循环自然退出 → 无环" 后置检查
                return None
            # 走到这说明有环
            if not fast or not fast.next:
                return None                 # 防御性 (被 break 出来的情况下才可能到)
            # Phase 2
            p = head
            while p is not slow:
                p = p.next
                slow = slow.next
            return p
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {ListNode}
     */
    var detectCycle = function(head) {
        let slow = head, fast = head;
        // Phase 1
        while (fast && fast.next) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow === fast) break;       // === 是 JS 严格相等, 引用比较跟 C++ 指针一致
        }
        if (!fast || !fast.next) return null;

        // Phase 2
        let p = head;
        while (p !== slow) {
            p = p.next;
            slow = slow.next;
        }
        return p;
    };
    ```

## Complexity

- **Time**: O(n) — Phase 1 ≤ 2n, Phase 2 ≤ n.
- **Space**: O(1) — 常数指针.

## 相关题目

- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap = n
- 0141\. Linked List Cycle (待补) — 只判是否有环 (Phase 1 就够)
- 0876\. Middle of the Linked List (待补) — 2×/1× 找中点
- 0287\. Find the Duplicate Number (待补) — **Floyd 判环用在数组上** — 神应用
- [0202. Happy Number](../../03-hash-table/0202-happy-number/README.md) — Floyd 判"数字循环"
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题
- 0234\. Palindrome Linked List (待补) — 快慢找中点 + 后半反转

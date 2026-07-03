# 0203. Remove Linked List Elements / 移除链表元素

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Linked List, Dummy Head, Pointer · 链表, 虚拟头节点, 指针
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-linked-list-elements/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Delete all nodes with `val`** → **dummy head** + single `cur` pointing to **predecessor**; on match, `cur->next = cur->next->next` (do **not** advance `cur`).
>
> **中文**: **删所有 val 节点** → **虚拟头节点** + `cur` 指**前驱**; 命中就 `cur->next = cur->next->next` (**不** 推进 cur).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给链表头 `head` 和 `val`. 删掉所有 `node.val == val` 的节点, 返回新头.

**中文**: 链表删值, 可能删头.

## Key Insights

1. **🔑 虚拟头节点 (dummy head) 是链表"删除/插入" 题的母模式 / Dummy head: the canonical pattern for delete/insert**

    朴素思路要**分两段**: "头节点要删" 单独处理 + "中间节点要删" 走一套指针操作. 代码丑且易错.

    **dummy 一加, 头节点变成"普通中间节点"** — 整段链表统一处理:

    ```
    dummy → head → ... → null
       ^
       cur (起点)
    ```

    `cur` 起在 dummy, 这样 `head` 本身也"有前驱可操作", 删除逻辑跟中间节点一模一样.

    > **凡是头节点可能被改 (删 / 插 / 替换), 都加 dummy**. 是链表题的"咒语级" 习惯.

2. **🔑 `cur` 指"前驱" 不是"当前节点" / `cur` points to the predecessor, not the current node**

    链表的删除**必须从前驱操作** — 因为单链表没法反查上一个. 所以遍历时:

    ```cpp
    while (cur->next) {                                      // 看的是"下一个"
        if (cur->next->val == val) ...                       // 判的也是"下一个"
        ...
    }
    ```

    Yang 用 `cur = &dummy` 起步, 全程 `cur->next` 是被检查的"目标节点", `cur` 自己是它的前驱.

    > **新手常见错**: 用 `cur->val == val` 判, 然后想"怎么删自己" → 死路. 必须**站在前驱看下一个**.

3. **🔑 删除时**不**推进 cur / Don't advance `cur` after deletion**

    删除后, `cur->next` 已经更新成了**再下一个节点*** (跳过被删的). 这时 cur 站在原位**继续判** — 若再下一个还是要删的, 一轮删一个直到不删.

    ```
    1 → 6 → 6 → 6 → 7   (val = 6)
    ^
    cur
    删 → 1 → 6 → 6 → 7  (cur 不动)
    删 → 1 → 6 → 7      (cur 还是不动)
    删 → 1 → 7
    不删 → cur 前进到 1
    ```

    若删除后**也推进** → 漏判紧跟着的同值节点. **删 = 原地, 不删 = 推进** — 经典的"或" 结构.

4. **🔑 C++ 栈上 dummy vs heap dummy / Stack vs heap dummy**

    Yang 用 `ListNode dummy(0); dummy.next = head;` — **栈上** 分配, 函数返回时自动析构.

    | 栈上 (Yang) | heap (`new ListNode(0)`) |
    |---|---|
    | `ListNode dummy(0);` + `dummy.next = head;` | `auto* dummy = new ListNode(0);` |
    | 返 `dummy.next` | 返 `dummy->next` (返前可 `delete dummy`) |
    | 自动清理, 零内存泄漏风险 | 必须手 delete, 不然泄漏 |
    | 写起来短 | 写起来啰嗦 |

    > **栈 dummy 是 C++ 链表题的最佳实践**. 没理由 new.

5. **🔑 `delete toDel` — 工程素养 / Manual cleanup**

    Yang 多写了 `delete toDel;` — LeetCode 不强制, 但**工程上不 delete 就是内存泄漏**. Python / JS 有 GC 不用管.

    > 面试时**写 delete 显示考虑过 ownership**. C++ 链表 = 手动管理.

6. **复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    一遍扫, 一个 cur 指针 + 一个 dummy. 跟节点数无关.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* removeElements(ListNode* head, int val) {
            ListNode dummy(0);                                       // 栈上 dummy, 自动析构
            dummy.next = head;
            ListNode* cur = &dummy;                                  // cur 站在前驱位

            while (cur->next) {
                if (cur->next->val == val) {
                    ListNode* toDel = cur->next;                     // 暂存被删节点
                    cur->next = cur->next->next;                     // 跳过它
                    delete toDel;                                    // 释放内存
                    // 注意: 不推进 cur, 继续判新的 cur->next
                } else {
                    cur = cur->next;                                 // 不删才推进
                }
            }
            return dummy.next;
        }
    };
    ```

=== "Python"
    ```python
    # Definition for singly-linked list.
    # class ListNode:
    #     def __init__(self, val=0, next=None):
    #         self.val = val
    #         self.next = next

    class Solution:
        def removeElements(self, head, val):
            # Python 没"栈节点", 但可以直接用 ListNode 实例当 dummy — GC 帮我们清理
            # 不用手 delete (C++ 的 delete) — Python refcount 归零自动回收
            dummy = ListNode(0, head)
            cur = dummy
            while cur.next:
                if cur.next.val == val:
                    cur.next = cur.next.next        # 直接跳过, GC 回收被删节点
                else:
                    cur = cur.next                  # 不删才前进
            return dummy.next
    ```

=== "JavaScript"
    ```javascript
    /**
     * Definition for singly-linked list.
     * function ListNode(val, next) {
     *     this.val = (val===undefined ? 0 : val)
     *     this.next = (next===undefined ? null : next)
     * }
     */
    /**
     * @param {ListNode} head
     * @param {number} val
     * @return {ListNode}
     */
    var removeElements = function(head, val) {
        // JS 同 Python: GC 接管, 不用 delete. 普通对象当 dummy
        // ListNode 构造可能不接收第二参数, 安全写法是显式赋值 next
        const dummy = new ListNode(0);
        dummy.next = head;
        let cur = dummy;
        while (cur.next) {
            if (cur.next.val === val) {
                cur.next = cur.next.next;       // 跳过被删节点
            } else {
                cur = cur.next;
            }
        }
        return dummy.next;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(1) — dummy + cur 两个指针.

## 相关题目

- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 链表反转, 迭代三指针母题
- [0707. Design Linked List](../0707-design-linked-list/README.md) — 自己实现链表, dummy + size 组合拳
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap = n + dummy
- 0021\. Merge Two Sorted Lists (待补) — dummy head 经典
- 0083\. Remove Duplicates from Sorted List (待补) — 类似删法
- 0082\. Remove Duplicates from Sorted List II (待补) — 删全部重复, 前驱判逻辑更绕
- 0237\. Delete Node in a Linked List (待补) — 没有前驱时怎么删 (复制下一节点的值)

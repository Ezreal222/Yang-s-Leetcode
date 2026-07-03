# 0206. Reverse Linked List / 反转链表

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Linked List, Iteration, Recursion · 链表, 迭代, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/reverse-linked-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Reverse singly linked list** → **iterative 3-pointer** (prev / curr / next): save next, flip `curr->next = prev`, march. Or **recursive**: reverse tail first, then `head->next->next = head; head->next = nullptr`.
>
> **中文**: **反转单链表** → **迭代三指针** (prev / curr / next): 存下家 → 翻转 → 推进. **递归**: 先反尾部, 再让下家反指回 head, 断掉 head 的旧 forward 指针.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给单链表头 `head`, 原地反转, 返回新头.

**中文**: 反转链表, 返回新头.

## Key Insights

1. **🔑 迭代三指针舞蹈: 链表操作的"母模板" / Iterative 3-pointer dance: the linked-list template**

    每步的**四步**顺序不能乱:

    ```
    ListNode* next = curr->next;    // 1. 备份 (不备就丢了)
    curr->next = prev;              // 2. 翻转当前节点
    prev = curr;                    // 3. prev 推进
    curr = next;                    // 4. curr 推进
    ```

    循环结束: `curr == nullptr`, `prev` 指向**原尾**  = **新头**.

    > **顺序为啥这样?** — 第 2 步会破坏 curr 原本 forward 指针 → 必须先备份 (第 1 步). 第 3, 4 步 prev / curr 各自往前一步. 记住"备→翻→推 prev→推 curr" 咒语.

2. **🔑 `next` 备份**必要**性 / Why save `next` first**

    若不备份:

    ```cpp
    curr->next = prev;              // 现在 curr->next == prev
    curr = curr->next;              // curr 变成 prev, 走错方向!
    ```

    **备份的本质**: 修改指针会同时"擦掉旧路", 想继续走旧路就得**先拍照**. 这是链表操作**通用**规则.

3. **🔑 递归: "信任子问题" (Recursive: trust the subproblem)**

    递归版的思维:

    - **假设**  `reverseList(head->next)` 会正确反转 head 后面所有节点, 返回新头.
    - 现在 head 的**下家 (head->next)** 变成了那段的**新尾巴** — 但 `head->next` 指针本身没变.
    - 我们要做的**只有两件事**:
        1. 让新尾巴反指回 head: `head->next->next = head;`
        2. 断掉 head 原来的 forward: `head->next = nullptr;` (否则 head ↔ 下家 死循环)
    - 返回上层传下来的 newHead (整个新链的头).

    ```
    head → A → B → C   (原)
    reverseList(A) 后:
    head → A ← B ← C   (A 还指着 head 因为没改)
              newHead = C
    执行 head->next->next = head:  即 A->next = head
    head ← A ← B ← C
    head->next 还指着 A, 会死循环 → 断掉 head->next = null
    A → B → C, A → head, head 是新尾. return newHead = C.
    ```

    > **递归解链表反转最考验"抽象信任"** — 想清楚"子问题返回什么" 这一句, 剩下就是补两行胶水.

4. **🔑 base case: 空或单节点 直接返 / Base case: empty or single**

    ```cpp
    if (!head || !head->next) return head;
    ```

    - `!head` — 空链表反转还是空.
    - `!head->next` — 只有一个节点, 反转 = 自己. 也是**递归停止的关键**: 一层层展开时, 最底层碰到"单节点" 就返回它 → 它就是**新头** newHead.

    > 少写一个条件 → 空链表 crash 或死递归. **两个都要**.

5. **🔑 迭代 vs 递归对比 / Iterative vs recursive**

    | | 迭代 (v1) | 递归 (v2) |
    |---|---|---|
    | 时间 | O(n) | O(n) |
    | **空间** | **O(1)** | **O(n)** (调用栈) |
    | 可读性 | 需要理解四步顺序 | 需要理解"信任子问题" |
    | 面试推荐 | **首选** | 展示"也会递归" |
    | 溢出风险 | 无 | 长链表可能 stack overflow |

    > **迭代是链表反转的正版答案**, 递归是"教学 / 面试知识展示". 生产代码基本都用迭代.

6. **🔑 返回值的语义 / Return value semantics**

    - **迭代版**: 循环结束 `curr == nullptr`, `prev` 就是**曾经的原尾** — 现在成新头. 返 `prev`.
    - **递归版**: `newHead` 是最底层返回的**原尾**, 一路透传上来. 返 `newHead`.

    > 两版**都是返"原尾"** — 因为原尾在反转后就是新头. 只是获取方式不同 (循环推 / 递归传).

7. **复杂度 O(n) 时间; 空间 O(1) 迭代 / O(n) 递归 / Linear time; O(1) iterative or O(n) recursive**

    每节点被访问一次. 空间差异见上表.

## Solution

=== "C++"

    **v1: 迭代三指针 (工程首选, O(1) 空间)**

    ```cpp
    class Solution {
    public:
        ListNode* reverseList(ListNode* head) {
            ListNode* prev = nullptr;
            ListNode* curr = head;
            while (curr) {
                ListNode* next = curr->next;                    // 备
                curr->next = prev;                              // 翻
                prev = curr;                                    // 推 prev
                curr = next;                                    // 推 curr
            }
            return prev;                                        // 循环结束 prev = 原尾 = 新头
        }
    };
    ```

    **v2: 递归 (教学向, O(n) 栈空间)**

    ```cpp
    class Solution {
    public:
        ListNode* reverseList(ListNode* head) {
            if (!head || !head->next) return head;              // base: 空 / 单节点
            ListNode* newHead = reverseList(head->next);        // 信任子问题: 后段已反转
            head->next->next = head;                            // 新尾反指回 head
            head->next = nullptr;                               // 断掉 head 旧 forward, 避免死循环
            return newHead;                                     // 一路透传原尾 = 新头
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1: 迭代 (推荐)
        def reverseList(self, head):
            # Python 没 nullptr, 用 None. 三指针语义完全对齐 C++
            prev, curr = None, head
            while curr:
                # 元组解包一行搞定四步. 右边全先算完 再赋左边, 天然避开"覆盖-丢失"
                # 等价 C++: next = curr.next; curr.next = prev; prev = curr; curr = next
                curr.next, prev, curr = prev, curr, curr.next
            return prev

        # v2: 递归
        def reverseList_rec(self, head):
            if not head or not head.next: return head
            new_head = self.reverseList_rec(head.next)
            head.next.next = head
            head.next = None
            return new_head
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
     * @return {ListNode}
     */
    var reverseList = function(head) {
        // v1: 迭代. JS 没解构式多重赋值到成员字段的等价物 (const [] = 只到局部变量),
        // 所以老实写四步. 用 let / null, 语义跟 C++ 一致
        let prev = null, curr = head;
        while (curr) {
            const next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    };

    // v2: 递归版 (备用)
    var reverseListRec = function(head) {
        if (!head || !head.next) return head;
        const newHead = reverseListRec(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问一次.
- **Space**: **O(1) 迭代** / O(n) 递归 (调用栈).

## 相关题目

- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — 链表基础, dummy head 母题
- [0707. Design Linked List](../0707-design-linked-list/README.md) — API 设计, 组合插/删
- 0092\. Reverse Linked List II (待补) — 部分反转 (给区间), 三指针 + dummy
- [0025. Reverse Nodes in k-Group](../0025-reverse-nodes-in-k-group/README.md) — 每 k 个一组反转, Hard
- [0024. Swap Nodes in Pairs](../0024-swap-nodes-in-pairs/README.md) — 每两个一组交换 = k=2 版
- 0234\. Palindrome Linked List (待补) — 快慢找中点 + 后半反转 + 比较
- 0143\. Reorder List (待补) — 快慢找中点 + 后半反转 + 双端 merge
- 0061\. Rotate List (待补) — 部分旋转, 指针操作

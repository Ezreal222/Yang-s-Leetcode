# 0024. Swap Nodes in Pairs / 两两交换链表节点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Dummy Head, Pointer · 链表, 虚拟头节点, 指针
    - **Link**: [LeetCode](https://leetcode.com/problems/swap-nodes-in-pairs/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Swap every two adjacent nodes** → **dummy + prev/a/b** three pointers; per pair: `a->next = b->next`, `b->next = a`, `prev->next = b`; advance `prev = a`.
>
> **中文**: **每两个节点交换** → **dummy + prev/a/b** 三指针; 每对: `a→next = b→next`, `b→next = a`, `prev→next = b`; 推进 `prev = a`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给单链表, 每**相邻两个节点交换** (不允许改 `val`, 只能改指针). 返回新头.

**中文**: 两两交换节点, 不能改值只能改指针.

## Key Insights

1. **🔑 又见 dummy: 头对交换 = 中间对交换 / Dummy again: unify head-pair with middle-pair swap**

    第一对 (head 和 head->next) 交换后, **头会变** → 想要"改前驱指向新头" 但**头没前驱** → 加 dummy 解决. 同 [0203](../0203-remove-linked-list-elements/README.md) / [0707](../0707-design-linked-list/README.md) 母模式.

    > **凡是链表操作会改头**, 立刻反射: **加 dummy**. 少一个特判.

2. **🔑 每对三指针 `prev / a / b` / Three pointers per pair**

    ```
    ... → prev → a → b → rest → ...
    ```

    Yang 命名: `prev` = 这对的前驱, `a` = 这对的第一个, `b` = 第二个. 交换后:

    ```
    ... → prev → b → a → rest → ...
    ```

    > **三指针每次锁定一对**, 递进往后走. 想清楚"要保存哪三个" 是链表舞蹈的关键抽象.

3. **🔑 三步交换的顺序 (先接后改) / Three-step swap: link forward first**

    ```cpp
    a->next = b->next;       // 1. a 跳过 b, 直连 rest (先把"未来的下家" 接上)
    b->next = a;             // 2. b 反指 a
    prev->next = b;           // 3. 前驱认新头 b
    ```

    **必须先算 `a->next = b->next`** — 因为下一步 `b->next` 就要被改, 不先接就丢了 rest. 跟 [0206](../0206-reverse-linked-list/README.md) 的"备份 next 再翻转" 同源思想.

    图解:

    ```
    初始:  prev → a → b → rest
    步 1:  prev → a → rest   (a 直接跳 b)
                   b → rest   (b 还悬着 next 指向 rest, 但没人指 b 了)
    步 2:  prev → a → rest
                   b → a → rest
    步 3:  prev → b → a → rest        ← 完成一对交换
    ```

4. **🔑 推进 `prev = a` / Advance `prev = a`**

    交换后, **原来的 a 现在是这对的尾** → 就是下对的前驱. 直接 `prev = a` 一步到位.

    > 若写 `prev = prev->next->next` 也对, 但没 `prev = a` 直白.

5. **🔑 循环条件 `prev->next && prev->next->next` / Loop guard: at least two more nodes**

    交换需要**两个完整节点** → 检查 `prev->next` (第一个存在) 和 `prev->next->next` (第二个存在). 顺序不能反 — 短路能防空指针解引用:

    - 若 `prev->next == null`, 第一个不存在 → `&&` 短路, 不算第二个. ✅
    - 若反过来 `prev->next->next && prev->next`, 第一个为 null 时右边先算 `null->next` = crash. ❌

    > **短路求值** 是 C++/Java/JS/Python 的宝藏 — 依赖它省一层判空.

6. **🔑 跟 [0206](../0206-reverse-linked-list/README.md) 反转的关系 / Relation to 0206**

    本题 = **k-group 反转** 的 k=2 特例. 通用版是 [0025 Reverse Nodes in k-Group](../../02-linked-list/index.md) (Hard, 待补). 熟悉了 k=2, k=3 / k=k 只是把"三指针" 扩到"k 指针 + 段内三指针舞".

    > 链表题的进阶路线: **反转全链 (0206) → 反转对 (0024) → k 组反转 (0025)**. 三级跳.

7. **复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    每节点被访问一次, 只有 dummy + 三个游标指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* swapPairs(ListNode* head) {
            ListNode dummy(0);                                       // 栈上 dummy
            dummy.next = head;
            ListNode* prev = &dummy;

            while (prev->next && prev->next->next) {                 // 至少还有一对
                ListNode* a = prev->next;
                ListNode* b = a->next;

                a->next = b->next;                                   // 1. 先接下家
                b->next = a;                                         // 2. b 反指 a
                prev->next = b;                                      // 3. 前驱认新头 b

                prev = a;                                            // 推进: a 成下对前驱
            }
            return dummy.next;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def swapPairs(self, head):
            # Python 无栈节点, 用 ListNode 实例. GC 兜底不用 delete
            dummy = ListNode(0, head)
            prev = dummy
            while prev.next and prev.next.next:      # 短路求值同 C++
                a, b = prev.next, prev.next.next
                a.next = b.next                       # 顺序仍然重要
                b.next = a
                prev.next = b
                prev = a
            return dummy.next

        # 递归版 (备用) — 4 行, 教学向
        def swapPairs_rec(self, head):
            if not head or not head.next: return head
            nxt = head.next
            head.next = self.swapPairs_rec(nxt.next)  # 信任子问题: 剩下已两两交换完
            nxt.next = head
            return nxt
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {ListNode}
     */
    var swapPairs = function(head) {
        // JS 普通对象当 dummy. { val, next } 短语法
        const dummy = { val: 0, next: head };
        let prev = dummy;
        while (prev.next && prev.next.next) {
            const a = prev.next;
            const b = a.next;
            a.next = b.next;
            b.next = a;
            prev.next = b;
            prev = a;
        }
        return dummy.next;
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问一次.
- **Space**: O(1) — dummy + 三个游标.

## 相关题目

- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转全链, 迭代母模板
- 0025\. Reverse Nodes in k-Group (待补, Hard) — 本题的推广: k=k 组反转
- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题
- [0707. Design Linked List](../0707-design-linked-list/README.md) — dummy + 指针综合
- 0086\. Partition List (待补) — dummy × 2 拆链再合并
- 1721\. Swapping Nodes in a Linked List (待补) — 只交换值 (更简单版)
- 0328\. Odd Even Linked List (待补) — dummy × 2 分组

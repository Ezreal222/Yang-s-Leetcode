# 0002. Add Two Numbers / 两数相加

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Math, Simulation, Dummy Head · 链表, 数学, 模拟, 虚拟头节点
    - **Link**: [LeetCode](https://leetcode.com/problems/add-two-numbers/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Two numbers as reversed-digit linked lists, return sum as another reversed list** → **dummy head + running carry**. Loop while **any of `l1`, `l2`, or `carry`** is non-zero: `sum = carry + (l1 ? l1.val : 0) + (l2 ? l2.val : 0)`, new node = `sum % 10`, carry = `sum / 10`.
>
> **中文**: **数字按倒序存链表, 求和也返倒序链表** → **dummy + 进位**. 循环条件三合一 (`l1 || l2 || carry`). 每步累加 + 建新节点 + 更新 carry.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 两个非负整数, 每个用链表**倒序** 表示 (个位在头). 求和, 返回**同样倒序**的链表.

- 例: `l1 = [2,4,3]` (=342), `l2 = [5,6,4]` (=465) → `[7,0,8]` (=807).

**中文**: 两个数字倒序存链表, 求和.

## Key Insights

1. **🔑 灵魂: 倒序存储让加法自然 / Reverse order = natural addition**

    数学加法**从个位开始**, 逐位相加带进位. 若数字**倒序** 存 (个位在头), **一遍扫**就能算 — 无需回溯.

    ```
    342 = [2,4,3]  → 从头 (个位 2) 开始加
    465 = [5,6,4]
    sum = 2+5=7 → 4+6=10 (carry 1) → 3+4+1=8
    结果 [7,0,8] = 807 ✓
    ```

    > **数据结构的顺序决定算法的方向**. 若题目**正序** 存 (百位在头, 如 0445), 就得先反转或用栈.

2. **🔑 循环条件三合一: `l1 || l2 || carry` / Three-condition loop**

    Yang 的核心一行:

    ```cpp
    while (l1 || l2 || carry) { ... }
    ```

    覆盖了**所有**结束场景:

    - 两链等长: 循环到都空 & carry = 0.
    - **不等长**: 长的还继续, 短的用 0 补.
    - **最高位有进位**: 例 `[5] + [5] = [0, 1]` — 循环最后一次是**只有 carry**, 新建 `[1]` 节点.

    > **少写 `|| carry`** 就漏最高位进位 case, 例 `999 + 1 = 1000` 会返 `[0, 0, 0]` 而非 `[0, 0, 0, 1]`. **一行代码涵盖 4 种 case**, 是这题的美感.

3. **🔑 `sum = carry` 起手, 干净 / Init sum with carry, then add**

    ```cpp
    int sum = carry;
    if (l1) { sum += l1->val; l1 = l1->next; }
    if (l2) { sum += l2->val; l2 = l2->next; }
    ```

    - **`sum = carry`** 天然继承上一步进位.
    - 两个独立 `if` 分别加 + 推进 — 优雅处理"一链已空另一链继续" 的情况.

    > **别写成 `sum = l1->val + l2->val + carry`** — 会 crash 当某链已空. Yang 的写法是防御性 + 简洁的平衡.

4. **🔑 `carry = sum / 10`, 值 = `sum % 10` / Standard carry decomposition**

    数字加法 max: 9 + 9 + 1 (前一进位) = **19**. → carry 最多 1, 位值最多 9.

    - `sum / 10` ∈ {0, 1} — 进位.
    - `sum % 10` ∈ {0..9} — 本位值.

    > "**div + mod** 拆分" 是数字类题的通用招. **加法只可能 carry 1**, 但**乘法**可能 carry 多 (见 0043 Multiply Strings).

5. **🔑 dummy head 又一次的用途 / Dummy head — again**

    没 dummy 就要特判"第一个新节点作为 head". 加 dummy → **每次都是"接在 curr 后面"**, 统一.

    > **链表建 / 改 → 加 dummy** 反射. 全家福: [0203](../0203-remove-linked-list-elements/README.md) / [0707](../0707-design-linked-list/README.md) / [0021](../0021-merge-two-sorted-lists/README.md) / [0024](../0024-swap-nodes-in-pairs/README.md) / [0019](../0019-remove-nth-node-from-end-of-list/README.md) / **本题**.

6. **🔑 建新节点 vs 复用原节点 / New nodes vs reuse**

    Yang 建**新节点** (`new ListNode(sum % 10)`) — **不 mutate 输入**. 更干净, 内存代价 O(n) 无所谓.

    可复用原节点省内存, 但会**破坏输入** — 面试通常**不推荐**.

7. **🔑 跟 [0415 Add Strings](../../04-string/0415-add-strings/README.md) / [0067 Add Binary](../../04-string/0067-add-binary/README.md) 一族 / Add family**

    | 题 | 数据结构 | 特点 |
    |---|---|---|
    | **0002 (本题)** | 链表, **倒序** | 从头加, 自然 |
    | 0415 Add Strings | 字符串, **正序** | 从**尾**加, `i--` |
    | 0067 Add Binary | 字符串, **正序** | 二进制, carry `/= 2` |
    | 0043 Multiply Strings (待补) | 字符串 | 乘法, carry 可 > 1 |
    | 0445 Add Two Numbers II (待补) | 链表, **正序** | 需**反转或用栈** |

    > **加法模拟 + 进位** 是数学题的核心模板, 数据结构决定"从头加还是从尾加".

8. **🔑 复杂度 O(max(m, n)) 时间/空间 / Linear in longer input**

    - Time: 一遍扫, 长度为 max(m, n) + 1 (最高进位).
    - Space: 新链表 O(max(m, n)) + 常数指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
            ListNode dummy(0);
            ListNode* curr = &dummy;
            int carry = 0;
            while (l1 || l2 || carry) {                              // 三合一
                int sum = carry;
                if (l1) { sum += l1->val; l1 = l1->next; }
                if (l2) { sum += l2->val; l2 = l2->next; }
                carry = sum / 10;
                curr->next = new ListNode(sum % 10);
                curr = curr->next;
            }
            return dummy.next;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def addTwoNumbers(self, l1, l2):
            dummy = ListNode(0)
            cur = dummy
            carry = 0
            while l1 or l2 or carry:
                s = carry
                if l1:
                    s += l1.val
                    l1 = l1.next
                if l2:
                    s += l2.val
                    l2 = l2.next
                # divmod(s, 10) 一步拿 (carry, val) — Pythonic
                # 等价 C++: carry = s / 10; val = s % 10
                carry, val = divmod(s, 10)
                cur.next = ListNode(val)
                cur = cur.next
            return dummy.next
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} l1
     * @param {ListNode} l2
     * @return {ListNode}
     */
    var addTwoNumbers = function(l1, l2) {
        const dummy = { val: 0, next: null };
        let cur = dummy;
        let carry = 0;
        while (l1 || l2 || carry) {
            let sum = carry;
            if (l1) { sum += l1.val; l1 = l1.next; }
            if (l2) { sum += l2.val; l2 = l2.next; }
            carry = Math.floor(sum / 10);       // JS `/` 是浮点, 要 Math.floor
            cur.next = { val: sum % 10, next: null };
            cur = cur.next;
        }
        return dummy.next;
    };
    ```

## Complexity

- **Time**: O(max(m, n)) — 一遍扫.
- **Space**: O(max(m, n)) — 新链表 (+1 for 最高进位).

## 相关题目

- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — dummy + 双指针 merge
- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢 gap
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题
- [0138. Copy List with Random Pointer](../0138-copy-list-with-random-pointer/README.md) — 复杂链表深拷贝
- [0415. Add Strings](../../04-string/0415-add-strings/README.md) — 字符串加法, 正序从尾加
- [0067. Add Binary](../../04-string/0067-add-binary/README.md) — 二进制加法
- 0369\. Plus One Linked List (待补) — 链表 +1, 需要反转或递归
- [0445. Add Two Numbers II](../0445-add-two-numbers-ii/README.md) — **正序** 链表加法, 反转 / 栈
- 0043\. Multiply Strings (待补) — 字符串乘法, carry 可 > 1
- 0989\. Add to Array-Form of Integer (待补) — 数组版加法

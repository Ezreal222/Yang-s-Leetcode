# 0445. Add Two Numbers II / 两数相加 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Stack, Math, Simulation · 链表, 栈, 数学, 模拟
    - **Link**: [LeetCode](https://leetcode.com/problems/add-two-numbers-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Two numbers as forward-order (MSB first) linked lists** → **stack both** (LIFO gives LSB first), then add with carry. **Head-insert new nodes** so the result comes out MSB-first — no final reverse needed.
>
> **中文**: **正序 (高位在头) 链表加法** → **两链入栈** (LIFO 天然从个位开始), 加法带进位. **头插结果** 让高位自动在头 — 免最后再反转.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 两个非负整数, 每个用链表**正序** 存 (**高位在头**). 求和, 返回**同样正序** 链表.

- 例: `l1 = [7,2,4,3]` (=7243), `l2 = [5,6,4]` (=564) → `[7,8,0,7]` (=7807).

**中文**: 正序数字链表相加. Follow-up: **不能修改输入**.

## Key Insights

1. **🔑 灵魂: 正序 vs 倒序 (跟 [0002](../0002-add-two-numbers/README.md) 差别) / Forward vs reverse order**

    - **[0002](../0002-add-two-numbers/README.md) 倒序** (个位在头): 从头加即从个位加 → 自然一遍扫.
    - **本题正序** (高位在头): 从头加是从**高位** 加 — 但加法必须从**个位** 开始!

    → 需要**反向访问** 两链. 三种破法:

    - **反转两链** (改输入) → 套 0002 → 反转结果. Follow-up 不允许改输入时不能用.
    - **栈** (LIFO 天然反转) — Yang 用这招, **不改输入**.
    - **递归** — 隐式用调用栈.

    > **"数据结构方向 vs 算法方向不一致 → 反转 or 栈"** 是通用招. Follow-up "不改输入" 就选栈.

2. **🔑 两链入栈: LIFO 天然从个位开始 / Stack gives LSB-first access**

    ```cpp
    stack<int> s1, s2;
    for (auto p = l1; p; p = p->next) s1.push(p->val);
    for (auto p = l2; p; p = p->next) s2.push(p->val);
    ```

    - **入栈顺序 = 链表顺序** (MSB → LSB).
    - **出栈顺序 = 反的** (LSB → MSB) — 正好对应"个位先算".
    - **不修改输入** — 只复制值, 原链保留.

    > **栈 = 反转的懒惰实现**. 想反着遍历但不能改原数据? 入栈.

3. **🔑 灵魂: 头插法建结果 (免最后再反转) / Head-insert to build result**

    Yang 的关键 3 行:

    ```cpp
    ListNode* node = new ListNode(sum % 10);
    node->next = head;              // 新节点指向当前 head
    head = node;                    // 新节点成为 head
    ```

    - 计算顺序: **个位 → 十位 → 百位** (低到高).
    - 我们要的结果顺序: **高位 → 个位** (正序).
    - **每次新节点作 head, 之前的 head 变第二个** → 新节点是**最高位** 的算出结果就成新 head → 最终 head = 最高位. ✓

    ```
    trace:
    计算个位  = 7: head = [7]
    计算十位  = 0: 新节点 [0]→[7], head = [0, 7]
    计算百位  = 8: 新节点 [8]→[0]→[7], head = [8, 0, 7]
    最高位   = 7: 新节点 [7]→[8]→[0]→[7], head = [7, 8, 0, 7] ✓
    ```

    > **头插法 = 边建边反转**. 免"先建 → 再 reverse" 两遍扫. 是这题的美感.

4. **🔑 循环条件同 [0002](../0002-add-two-numbers/README.md): `s1 || s2 || carry` / Same 3-cond loop**

    ```cpp
    while (!s1.empty() || !s2.empty() || carry) { ... }
    ```

    - 两栈不等大: 弹空的用 0 补.
    - **`carry` 最后可能新生成一个最高位节点** (例 `[5]+[5]=[1,0]`).

    跟 0002 一样, **少 `carry` 就漏最高位进位**.

5. **🔑 `sum / 10, sum % 10` — 标准加法拆分 / Standard div+mod**

    - carry ∈ {0, 1} (数字加法 max 9+9+1=19).
    - val ∈ {0..9}.

    跟 0002 完全一样.

6. **🔑 备选: 反转链表版 / Alt: reverse-based**

    ```
    l1 = reverse(l1)
    l2 = reverse(l2)
    result = addTwoNumbers(l1, l2)   // 用 0002 的解
    return reverse(result)
    ```

    - O(n) 时间, **O(1) 额外空间** (若允许改输入).
    - **改了输入** — Follow-up "不改输入" 就不能这样.
    - 若还原输入, 结束前得再反转两链, 6 次遍历太累.

    > **栈版 vs 反转版**: 栈**多 O(n) 空间但不改输入**; 反转**省空间但改输入**. Follow-up 明确要求不改就上栈.

7. **🔑 跟 [0002](../0002-add-two-numbers/README.md) 组成加法家族 / Add family**

    | | [0002 Add Two Numbers](../0002-add-two-numbers/README.md) | **0445 (本题)** |
    |---|---|---|
    | 存储顺序 | **倒序** (LSB → MSB) | **正序** (MSB → LSB) |
    | 遍历方向 | 从头顺序加 | **需反向** (栈 / 反转) |
    | 建结果 | 尾部追加 (dummy) | **头插** (免反转) |
    | 改不改输入 | 不改 | Follow-up 不改 |

    > **一字之差**, 数据结构方向变了, **算法两处都变** (输入访问 + 输出建立). 记熟这对镜像.

8. **🔑 复杂度 O(m + n) 时间/空间 / Linear**

    - Time: 两栈建 + 一遍加.
    - Space: 两栈 O(m + n) + 结果 O(max(m, n) + 1).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
            stack<int> s1, s2;
            for (ListNode* p = l1; p; p = p->next) s1.push(p->val);
            for (ListNode* p = l2; p; p = p->next) s2.push(p->val);
            int carry = 0;
            ListNode* head = nullptr;
            while (!s1.empty() || !s2.empty() || carry) {
                int sum = carry;
                if (!s1.empty()) { sum += s1.top(); s1.pop(); }
                if (!s2.empty()) { sum += s2.top(); s2.pop(); }
                ListNode* node = new ListNode(sum % 10);
                carry = sum / 10;
                node->next = head;                                    // 头插 — 免最后反转
                head = node;
            }
            return head;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def addTwoNumbers(self, l1, l2):
            # list 当栈, .pop() 从尾 O(1) — Python 列表天然是栈
            s1, s2 = [], []
            while l1: s1.append(l1.val); l1 = l1.next
            while l2: s2.append(l2.val); l2 = l2.next
            carry = 0
            head = None
            while s1 or s2 or carry:
                s = carry
                if s1: s += s1.pop()
                if s2: s += s2.pop()
                carry, val = divmod(s, 10)              # 一步拿 (进位, 位值)
                node = ListNode(val)
                node.next = head                         # 头插
                head = node
            return head
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} l1
     * @param {ListNode} l2
     * @return {ListNode}
     */
    var addTwoNumbers = function(l1, l2) {
        const s1 = [], s2 = [];
        for (let p = l1; p; p = p.next) s1.push(p.val);
        for (let p = l2; p; p = p.next) s2.push(p.val);
        let carry = 0, head = null;
        while (s1.length || s2.length || carry) {
            let sum = carry;
            if (s1.length) sum += s1.pop();
            if (s2.length) sum += s2.pop();
            carry = Math.floor(sum / 10);
            const node = { val: sum % 10, next: head };  // 一步头插 (对象字面量)
            head = node;
        }
        return head;
    };
    ```

## Complexity

- **Time**: O(m + n) — 两次入栈 + 一次加法.
- **Space**: O(m + n) — 两个栈.

## 相关题目

- [0002. Add Two Numbers](../0002-add-two-numbers/README.md) — **倒序版母题**, 本题的镜像
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题 (备选方案基础)
- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — dummy + merge
- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy 母题
- [0415. Add Strings](../../04-string/0415-add-strings/README.md) — 字符串正序, 从尾加 (类似)
- [0067. Add Binary](../../04-string/0067-add-binary/README.md) — 二进制加法
- 0369\. Plus One Linked List (待补) — 链表 +1
- 0043\. Multiply Strings (待补) — 字符串乘法
- 0989\. Add to Array-Form of Integer (待补) — 数组版加法

# 0430. Flatten a Multilevel Doubly Linked List / 扁平化多级双向链表

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, DFS, Doubly Linked List, Recursion · 链表, DFS, 双向链表, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Flatten DLL with nested children into single level** → **DFS returning tail**: on `child`, recurse to flatten sub-list (returns its tail), stitch `curr ↔ child`, `childTail ↔ next`, null out `child`, advance to saved `next`.
>
> **中文**: **多级双向链表拉平** → **DFS 返回子段尾**: 遇 child 就递归拉平 (返尾), 缝合 `curr ↔ child` + `childTail ↔ next`, 清 child 指针, 推进到备份的 next.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 双向链表, 每个节点可能有 `child` 指向另一条子链. **拉平**成单层双向链表: 遇 child 就把整条子链插入 curr 和 curr->next 之间, 清掉 child 指针.

**中文**: 双向链表带子链, 拉平成单层. 子链插入原位, child 清空.

## Key Insights

1. **🔑 API 设计: 递归返回**tail** / API design: recurse returns tail**

    Yang 的**关键选择**: helper `flattenDFS` **返回子段拉平后的尾**. 为啥不返 head?

    - **拉平 child 段** 后, 需要把它接到 curr 和 next 之间 → 需要**尾** 才能接 next.
    - 若只返 head, 得再扫一遍找 tail → O(n²) 累积.

    > **设计递归 API 时问一句"父调用需要子调用返回什么信息"** — 这里父需要 tail (缝合下家), 就返 tail. 类似思想: 后序遍历里返多个值 (0110, 0104).

2. **🔑 遇 child 的 5 步舞蹈 / The 5-step dance on child**

    ```cpp
    Node* next = curr->next;                   // 1. 备份 next
    Node* childTail = flattenDFS(curr->child); // 2. 递归拉平 child 段, 拿尾

    curr->next = curr->child;                  // 3a. curr → child head (向前)
    curr->child->prev = curr;                  // 3b. child head → curr (向后)
    curr->child = nullptr;                     // 3c. 清 child 指针

    if (next) {
        childTail->next = next;                // 4a. child tail → next
        next->prev = childTail;                // 4b. next → child tail
    }
    ```

    - 步 1 备份是必须的 — 步 3a 就要改 `curr->next`.
    - **双向链表要双向接指针** — 每处接口都两下.
    - **步 3c 清 child** — 拉平后子链已合入主链, child 指针必须清零 (题目要求).
    - **步 4 的 if** — next 可能是 null (curr 是原尾), 那就没啥可接的.

3. **🔑 推进永远是 `curr = next` 不是 `curr = child` / Advance to saved `next`, always**

    虽然 curr 后面接了 child 段, **推进不需要走完 child** — 因为 child 段**内部** 已被递归拉平了 (递归天然处理嵌套). 直接跳到原 next 继续.

    > **递归的力量**: 你不用管子问题**内部**, 只管**接口**. 主循环只推进 next, 子链处理归子调用.

4. **🔑 `last` 追踪当前段尾 / `last` tracks the tail of *this* segment**

    每次迭代结束更新 last:

    - 无 child: `last = curr`
    - 有 child: `last = childTail` (即拉平后 child 段的尾, 也是这段的当前尾)

    循环结束: `last` 就是**当前调用负责的整段** 的尾, 返给上层.

    > 这是"返回 tail" 契约的**兑现**. 想清楚 last 语义, 递归就通了.

5. **🔑 双向链表的两下功夫 / DLL: two links per edge**

    单向链表接 A → B 只写 `A.next = B`; 双向必写:

    ```cpp
    A->next = B;
    B->prev = A;
    ```

    **漏一下**就断链. 本题**每处接口都两下** (curr ↔ child head, child tail ↔ next). 检查代码时**两两对应看**.

6. **🔑 迭代版 (stack, 空间稍差可读) / Iterative alternative**

    用 stack 存"待处理的 next" — 遇 child 时把 next 压栈, 转向 child. 空间 O(depth), 跟递归调用栈同阶. **代码稍长**, 面试写递归就够.

7. **复杂度 O(n) 时间, O(depth) 空间 / Linear time, depth-space**

    每节点被访问 1 次. 递归深度 = 最深嵌套层数 (worst case n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        Node* flatten(Node* head) {
            flattenDFS(head);
            return head;
        }
    private:
        // 返回: 拉平后这段的 tail
        Node* flattenDFS(Node* head) {
            Node* curr = head;
            Node* last = nullptr;

            while (curr) {
                Node* next = curr->next;                            // 1. 备份 next (curr->next 会被改)

                if (curr->child) {
                    Node* childTail = flattenDFS(curr->child);      // 2. 递归拉平子段, 拿尾

                    // 3. curr ↔ child head (双向接)
                    curr->next = curr->child;
                    curr->child->prev = curr;
                    curr->child = nullptr;                          // 清 child

                    // 4. childTail ↔ next (若有)
                    if (next) {
                        childTail->next = next;
                        next->prev = childTail;
                    }

                    last = childTail;
                } else {
                    last = curr;
                }

                curr = next;                                        // 推进
            }
            return last;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def flatten(self, head: 'Node') -> 'Node':
            self._flatten_dfs(head)
            return head

        def _flatten_dfs(self, head):
            # 返: 这段的 tail. 跟 C++ 完全同源
            curr, last = head, None
            while curr:
                nxt = curr.next
                if curr.child:
                    child_tail = self._flatten_dfs(curr.child)
                    # curr ↔ child head
                    curr.next = curr.child
                    curr.child.prev = curr
                    curr.child = None                # None 相当于 C++ nullptr
                    # childTail ↔ next
                    if nxt:
                        child_tail.next = nxt
                        nxt.prev = child_tail
                    last = child_tail
                else:
                    last = curr
                curr = nxt
            return last
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {Node} head
     * @return {Node}
     */
    var flatten = function(head) {
        const dfs = (head) => {
            let curr = head, last = null;
            while (curr) {
                const next = curr.next;
                if (curr.child) {
                    const childTail = dfs(curr.child);
                    curr.next = curr.child;
                    curr.child.prev = curr;
                    curr.child = null;
                    if (next) {
                        childTail.next = next;
                        next.prev = childTail;
                    }
                    last = childTail;
                } else {
                    last = curr;
                }
                curr = next;
            }
            return last;
        };
        dfs(head);
        return head;
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问 1 次.
- **Space**: O(depth) — 递归栈深度 = 嵌套层数 (最坏 n).

## 相关题目

- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题, 递归与迭代对照
- [0024. Swap Nodes in Pairs](../0024-swap-nodes-in-pairs/README.md) — dummy + 三指针
- [0025. Reverse Nodes in k-Group](../0025-reverse-nodes-in-k-group/README.md) — 半开区间 + helper 返值
- 0114\. Flatten Binary Tree to Linked List (待补) — 树版拉平, 同款"返 tail" 递归
- 0138\. Copy List with Random Pointer (待补) — 复杂指针复制
- [0426. Convert BST to Sorted Doubly Linked List](../0426-convert-bst-to-sorted-doubly-linked-list/README.md) — 双向链表 + 中序
- 0708\. Insert into a Sorted Circular Linked List (待补) — 循环双向链表

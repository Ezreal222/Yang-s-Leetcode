# 1019. Next Greater Node In Linked List / 链表中的下一个更大节点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Linked List · 单调栈, 链表
    - **Link**: [LeetCode](https://leetcode.com/problems/next-greater-node-in-linked-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给链表 `head`. 对每个节点 i, 返回**链表中其右侧第一个比它大** 的节点值. 没有则 `0`.

**中文**: 链表上跑 NGE, 返回每个位置的下一个更大值, 没有 0.

## Key Insights

1. **🔑 跟 [0739](../0739-daily-temperatures/README.md) / [0496](../0496-next-greater-element-i/README.md) 同模板, 难点是"链表不支持随机访问" / Same NGE template, harder data structure**

    单调栈模板没变. 难点是**链表没有索引**, 不能像数组那样用 `arr[stk.top()]` 取值. 两种应对:

    | 解法 | 思路 | 复杂度 | 优缺 |
    |---|---|---|---|
    | **v1**: 转数组再跑 | 先 `cur = head` 扫一遍存进 vector, 再标准 NGE | O(n) 时间, O(n) 额外空间 | 简单直接, 两遍 |
    | **v2**: 一遍扫 + 栈存 (val, ans 索引) | 边走链表边压栈, 栈元素同时携带"值" 和"该值答案要写到 ans 哪个位置" | O(n) 时间, O(n) 栈 (无额外数组) | 紧凑, 一遍 |

    > **栈存什么** 看后续要做什么. v2 既要比较 (要 val) 又要更新答案 (要索引), 所以存 `pair`. 跟 [0901](../0901-online-stock-span/README.md) 栈存 `(price, span)` 是同思想 — **栈元素是"信息块"** 而非单值.

2. **v1 转数组解法 — 最直观 / Convert to array, classical NGE**

    扫链表把所有 val 存进 vector, 然后跑跟 [0739](../0739-daily-temperatures/README.md) 一字不差的 NGE (栈存索引, `res[stk.top()] = vec[i]`).

    > 没什么巧, 但**简单可读**. 链表本身的"无索引" 缺陷被转数组绕过.

3. **v2 一遍解法 — 边走边压, 索引提前分配 / One-pass with placeholder**

    遍历链表时**先 push_back(0) 占位** — 节点 i 还没找到答案, 先把它的答案位置留出来. 栈里存 `(val, 这个占位的索引)`. 后面更大值来时, 直接 `ans[索引] = newVal`.

    > **占位 + 延迟填充** 是流式题的标准技巧, 跟 [0901](../0901-online-stock-span/README.md) 设计上同精神.

4. **`0` 默认值兜底 — "没有更大" 不需要特殊处理 / Default 0 means "no next greater"**

    `ans.push_back(0)` 或 `vector<int> res(n, 0)` 起手. 找到更大值就覆盖, 找不到的留 0. 跟 [0503](../0503-next-greater-element-ii/README.md) 默认 -1 是镜像 — **题目约定决定缺省值**.

5. **复杂度 O(n) — 两版都是 / Both O(n)**

    每节点最多入栈一次出栈一次. v1 多一次扫链表填数组, 仍 O(n).

## Solution

=== "C++"
    === "v1: 转数组再跑标准 NGE"
        ```cpp
        class Solution {
        public:
            vector<int> nextLargerNodes(ListNode* head) {
                vector<int> vec;
                for (ListNode* cur = head; cur; cur = cur->next) vec.push_back(cur->val);
                int n = vec.size();
                vector<int> res(n, 0);
                stack<int> stk;                                    // 栈存索引
                for (int i = 0; i < n; i++) {
                    while (!stk.empty() && vec[i] > vec[stk.top()]) {
                        res[stk.top()] = vec[i];
                        stk.pop();
                    }
                    stk.push(i);
                }
                return res;
            }
        };
        ```

    === "v2 推荐: 一遍扫 + 栈存 (val, ans 索引)"
        ```cpp
        class Solution {
        public:
            vector<int> nextLargerNodes(ListNode* head) {
                vector<int> ans;
                stack<pair<int, int>> st;                          // (value, index in ans)
                int i = 0;
                for (ListNode* p = head; p; p = p->next, i++) {
                    ans.push_back(0);                              // 占位, 默认 0
                    while (!st.empty() && p->val > st.top().first) {
                        ans[st.top().second] = p->val;             // 找到了, 填进对应位置
                        st.pop();
                    }
                    st.push({p->val, i});
                }
                return ans;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def nextLargerNodes(self, head: ListNode) -> list[int]:
            ans = []
            stk = []                                               # [(val, idx), ...]
            i = 0
            cur = head
            while cur:
                ans.append(0)                                      # 占位
                while stk and cur.val > stk[-1][0]:
                    _, idx = stk.pop()
                    ans[idx] = cur.val
                stk.append((cur.val, i))
                cur = cur.next
                i += 1
            return ans
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {number[]}
     */
    var nextLargerNodes = function(head) {
        const ans = [];
        const stk = [];                                            // [[val, idx], ...]
        let i = 0;
        for (let p = head; p; p = p.next, i++) {
            ans.push(0);
            while (stk.length && p.val > stk[stk.length - 1][0]) {
                const [, idx] = stk.pop();
                ans[idx] = p.val;
            }
            stk.push([p.val, i]);
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) — vector + 栈.

## 相关题目

- [0739. Daily Temperatures](../0739-daily-temperatures/README.md) — 数组上的 NGE, 同款模板
- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — NGE 最基础
- [0503. Next Greater Element II](../0503-next-greater-element-ii/README.md) — 循环数组 NGE
- [0901. Online Stock Span](../0901-online-stock-span/README.md) — 同款"栈存信息块" 的设计思想
- 0206\. Reverse Linked List (待补) — 链表预处理: 反转后从后往前算 NGE 也行
- 0707\. Design Linked List (待补) — 链表设计基础

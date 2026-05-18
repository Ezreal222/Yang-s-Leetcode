# 0109. Convert Sorted List to Binary Search Tree / 有序链表转换二叉搜索树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, Linked List, Recursion, Divide and Conquer · 二叉树, 二叉搜索树, 链表, 递归, 分治
    - **Link**: [LeetCode](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a sorted singly linked list, build a **height-balanced** BST.

**中文**: 给一个有序单链表, 构造一棵**高度平衡**的 BST.

## 思路

### Core idea

跟 [0108 (sorted array → BST)](../0108-convert-sorted-array-to-binary-search-tree/README.md) 同款"取中点当根"骨架, 难点是**链表没有 O(1) 随机访问**, 找中点变成主成本. 有三个版本, 时间复杂度从 O(n log n) 一路压到 O(n).

### Key Insights

1. **数组版的 O(1) 取中点在链表里变 O(n) / Midpoint cost flips**

    [0108](../0108-convert-sorted-array-to-binary-search-tree/README.md) 数组中点是 `nums[mid]`, O(1). 链表中点要走一遍 (快慢指针或先数长度), O(子树大小). 朴素 fast/slow 法在每层都 O(n), 共 O(log n) 层 → 总 **O(n log n)**.

2. **方法 C: BST 中序 = 链表顺序 / Inorder of constructed BST consumes the list left-to-right**

    最妙的优化. 思路: 把构造和遍历**对齐**.

    - 先 O(n) 数出链表长度 `n`, 然后 build 用**下标参数** (不是真的链表指针!) 模拟"在下标空间二分".
    - 中序构造: **先**递归造左子树, **然后**才用 `cur->val` 创建当前节点并 **`cur = cur->next`** 前进, **再**造右子树.
    - 因为 BST 中序产出升序值, 而链表本来就升序, **构造的中序节点访问顺序 = 链表前进顺序**, 一个全局 `cur` 指针顺着扫一遍即可.

    每个链表节点只被访问一次 (创建那次), 总 **O(n)** —— 比方法 B 快了一个 log.

3. **半开区间 vs 双索引 / Interval styles**

    方法 B Yang 用了**半开** `[head, tail)`, base case `head == tail`. 同 0105/0106/0654, 不同于 0108 的闭区间. 半开在链表 + 快慢指针组合下很自然 (`tail = nullptr` 就是入口, 子调用 `tail = slow`).

    方法 C 用**闭区间** `[left, right]`, 跟 0108 一致 —— 因为这里下标是绝对位置.

### 一句话总结

**链表版的"取中点当根". 朴素法 (B): 快慢指针每层 O(n) 找中点, 共 O(n log n). 优化法 (C): 用下标递归 + 全局 `cur` 指针, **中序顺序构造**让 cur 顺着链表扫一遍, O(n).**

## Solution

### Variant A — convert to array, then call 0108

最直接 + 偷懒. 用 `std::vector`/`list`/`array` 把链表抄一遍, 然后跑 [0108](../0108-convert-sorted-array-to-binary-search-tree/README.md). 时间 O(n), 空间多 O(n). 不展开代码, 把 0108 拿来即可.

### Variant B — fast/slow midpoint, O(n log n)

=== "C++"
    ```cpp
    class Solution {
    public:
        // 半开区间 [head, tail)
        TreeNode* build(ListNode* head, ListNode* tail) {
            if (head == tail) return nullptr;
            // 快慢指针找中点
            ListNode* slow = head;
            ListNode* fast = head;
            while (fast != tail && fast->next != tail) {
                slow = slow->next;
                fast = fast->next->next;
            }
            TreeNode* root = new TreeNode(slow->val);
            root->left  = build(head, slow);
            root->right = build(slow->next, tail);
            return root;
        }
        TreeNode* sortedListToBST(ListNode* head) {
            return build(head, nullptr);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sortedListToBST(self, head: 'ListNode | None') -> 'TreeNode | None':
            def build(h, t):
                if h is t:
                    return None
                slow, fast = h, h
                while fast is not t and fast.next is not t:
                    slow = slow.next
                    fast = fast.next.next
                root = TreeNode(slow.val)
                root.left  = build(h, slow)
                root.right = build(slow.next, t)
                return root
            return build(head, None)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @return {TreeNode}
     */
    var sortedListToBST = function(head) {
        const build = (h, t) => {
            if (h === t) return null;
            let slow = h, fast = h;
            while (fast !== t && fast.next !== t) {
                slow = slow.next;
                fast = fast.next.next;
            }
            const root = new TreeNode(slow.val);
            root.left  = build(h, slow);
            root.right = build(slow.next, t);
            return root;
        };
        return build(head, null);
    };
    ```

### Variant C — inorder simulation, O(n) 最优

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* cur;   // 全局: 当前正在消费的链表位置
        int getLength(ListNode* head) {
            int len = 0;
            while (head) { len++; head = head->next; }
            return len;
        }
        // [left, right] 闭区间, 在"下标空间"分治
        TreeNode* build(int left, int right) {
            if (left > right) return nullptr;
            int mid = left + (right - left) / 2;
            // 中序: 先建左子树 (它会消费链表前段)
            TreeNode* leftChild = build(left, mid - 1);
            // 此刻 cur 恰好指向"中点应有的值"
            TreeNode* root = new TreeNode(cur->val);
            cur = cur->next;             // 消费掉, 前进
            root->left  = leftChild;
            root->right = build(mid + 1, right);
            return root;
        }
        TreeNode* sortedListToBST(ListNode* head) {
            cur = head;
            int n = getLength(head);
            return build(0, n - 1);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sortedListToBST(self, head: 'ListNode | None') -> 'TreeNode | None':
            # 先数长度
            n = 0
            p = head
            while p:
                n += 1
                p = p.next

            self.cur = head
            def build(lo: int, hi: int) -> 'TreeNode | None':
                if lo > hi:
                    return None
                mid = (lo + hi) // 2
                left = build(lo, mid - 1)        # 左子树先消费链表前段
                node = TreeNode(self.cur.val)
                self.cur = self.cur.next         # 顺序消费
                node.left  = left
                node.right = build(mid + 1, hi)
                return node
            return build(0, n - 1)
    ```

=== "JavaScript"
    ```javascript
    var sortedListToBST = function(head) {
        let n = 0;
        for (let p = head; p; p = p.next) n++;

        let cur = head;
        const build = (lo, hi) => {
            if (lo > hi) return null;
            const mid = (lo + hi) >> 1;
            const left = build(lo, mid - 1);     // 左子树消费链表前段
            const node = new TreeNode(cur.val);
            cur = cur.next;
            node.left  = left;
            node.right = build(mid + 1, hi);
            return node;
        };
        return build(0, n - 1);
    };
    ```

## Complexity

| | Time | Space (excl. output) |
|---|---|---|
| A. array + 0108 | **O(n)** | O(n) array |
| B. fast/slow midpoint | **O(n log n)** | O(log n) recursion |
| C. inorder simulation | **O(n)** | O(log n) recursion |

C 是面试满分解, A 是面试合格解, B 是教科书解.

## 易错点

- **方法 B 半开 `[head, tail)` 的快慢条件**: `while (fast != tail && fast->next != tail)`. 写成 `while (fast && fast->next)` 不行 —— 当 tail 不是 nullptr 时, 链表实际未结束, 但子调用的边界不再是真 nullptr.
- **方法 C 必须**先**建左子树**: 不能写成"先建当前 + 推进 cur + 建左". 那样 cur 在建左之前就已经走过中点了, 左子树拿不到正确的链表段. 顺序必须严格中序: **左 → 中 (含推进) → 右**.
- **方法 C 的 `cur` 是全局 / 闭包**: Python/JS 用闭包 / 实例属性; C++ 用类成员. 别写成局部变量 —— 递归不共享 cur 就废了.
- **空链表**: 入口 `head == nullptr` 时 n = 0, `build(0, -1)` 触发 `lo > hi` 返 null. 自然处理.
- **方法 B 选 slow 还是 slow->next 当根**: 题目允许任一合法平衡形状, 两种都行. 上面取 slow.

## 相关题目

- [0108. Convert Sorted Array to BST](../0108-convert-sorted-array-to-binary-search-tree/README.md) — 数组版, 这题的"前置准备题"
- [0105. Construct Binary Tree from Preorder and Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 同款分治构造
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 提供"BST 中序 = 升序"的核心性质
- [0098. Validate BST](../0098-validate-binary-search-tree/README.md) — 也用 inorder + cur 指针的模式
- 0876. Middle of the Linked List (待补) — 快慢指针找链表中点入门
- 1382. Balance a BST (待补) — 复合: 先 inorder dump 成 array, 再用 0108 重建

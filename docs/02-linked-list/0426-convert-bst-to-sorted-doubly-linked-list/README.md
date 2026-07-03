# 0426. Convert Binary Search Tree to Sorted Doubly Linked List / 将二叉搜索树转化为排序的双向链表

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, BST, Inorder DFS, Doubly Linked List · 链表, 二叉搜索树, 中序遍历, 双向链表
    - **Link**: [LeetCode](https://leetcode.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **BST → circular sorted DLL in-place** → **inorder DFS** (naturally sorted) + global `head` / `prev`; per visit: `prev.right = curr; curr.left = prev; prev = curr`; after DFS close the ring: `head.left = prev; prev.right = head`.
>
> **中文**: **BST 原地转有序循环双向链表** → **中序 DFS** (天然升序) + 全局 `head` / `prev`; 每访问一个节点接双向指针; 结束时首尾相连成环.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 BST 根. **原地** 转成**循环排序双向链表** (每个 Node 有 left/right, 复用为 prev/next). 返回 head (最小值).

**中文**: BST 就地转有序循环双向链表, 用 left / right 当 prev / next.

## Key Insights

1. **🔑 灵魂观察: BST 中序 = 升序序列 / BST inorder = sorted order**

    BST 的定义天然保证: 中序遍历得到**从小到大** 的节点流. 于是问题变成"**给一个有序节点流, 构造双向链表**" — 就是链表基本操作.

    > **看到 BST + "有序 / 第 k 大 / 相邻差"** → 反应中序. 是 §07 的核心洞察, 在这题被复用.

2. **🔑 全局 `head` + `prev` 就地缝合 / Global `head` + `prev` stitch as we go**

    Yang 用两个成员变量:

    - `head` — 最终返回的首节点 (最小值节点), 只在**第一次访问** 时设.
    - `prev` — 上一个被访问的节点. 当前节点访问时用来接指针.

    每次 inorder 中"访问 curr":

    ```cpp
    if (prev) {
        prev->right = curr;     // 双向接: 前 → 后
        curr->left = prev;      // 双向接: 后 → 前
    } else {
        head = curr;            // 第一个访问的就是 head (最左叶)
    }
    prev = curr;                // 推进
    ```

    > **inorder 里的"访问" 就是"把 curr 挂到 DLL 尾"**. `prev` 是"当前 DLL 的尾". 一路挂下去, DLL 一直是有序的.

3. **🔑 inorder DFS 三步模板 / Standard inorder template**

    ```cpp
    void inorder(Node* curr) {
        if (!curr) return;
        inorder(curr->left);       // 左
        // 访问 curr (本题在此接双向指针)
        inorder(curr->right);      // 右
    }
    ```

    - **左 → 中 → 右** 顺序保证升序.
    - 递归而非迭代 — 代码 4 行, 栈深 O(h).

    > 中序模板跟 [0094](../../07-binary-tree/0094-binary-tree-inorder-traversal/README.md) 完全一致. 这题只在"访问" 处塞了缝合逻辑.

4. **🔑 head 只在第一次赋一次 / `head` set once at leftmost**

    `if (prev) ... else head = curr;` — `prev` 是 null 表示"我们还没访问过任何节点" → 当前是**中序里的第一个** = **BST 里的最左叶**. 记录为 head.

    之后所有节点访问时 `prev` 都非 null → 走 stitch 分支.

    > **`prev == null` 是"第一次" 的天然标志**. 少一个 boolean flag.

5. **🔑 循环关闭: 遍历结束后 head ↔ prev / Close the ring after DFS**

    遍历完 `prev` 变成"最后一个访问的节点" = **DLL 的尾** (最大值). 题目要**循环双向链表** → 首尾相连:

    ```cpp
    head->left = prev;         // 首 → 尾 (环的一边)
    prev->right = head;        // 尾 → 首 (环的另一边)
    ```

    **两下**, 完成环. 若忘了这两行 → 得到"开放式" DLL, 不合规.

6. **🔑 复用 left/right 作 prev/next 的隐含契约 / Reusing left/right as prev/next**

    Node 的 left/right **在 BST 里** 是子指针, **转 DLL 后** 语义变成 prev/next. 题目允许 — 就地转换, 不额外分配.

    > 若不允许改原树, 得 clone → 额外 O(n) 空间. **原地** 是题目"节省内存" 的巧妙契约.

7. **🔑 迭代版 + Morris (面试进阶) / Iterative alternatives**

    - **迭代 + 栈**: 用栈显式模拟 inorder, 每弹一个就接 DLL. O(h) 空间.
    - **Morris**: 就地用节点的 right 空指针做"线索", **O(1) 空间**. 代码复杂, 面试知道就行.

    > 递归版**够用 + 好读**. 面试若问 O(1) 空间 → 提 Morris.

8. **复杂度 O(n) 时间, O(h) 空间 / Linear time, height space**

    每节点访问 1 次. 递归栈 = 树高 (平衡 O(log n), 极端 O(n)).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        Node* treeToDoublyList(Node* root) {
            if (!root) return nullptr;
            head = nullptr;
            prev = nullptr;
            inorder(root);
            head->left = prev;                       // 关闭环: 首 → 尾
            prev->right = head;                      // 尾 → 首
            return head;
        }
    private:
        Node* head;
        Node* prev;

        void inorder(Node* curr) {
            if (!curr) return;
            inorder(curr->left);                     // 左

            // 中: 访问 curr, 挂到 DLL 尾
            if (prev) {
                prev->right = curr;
                curr->left = prev;
            } else {
                head = curr;                         // 第一次 → 记 head
            }
            prev = curr;                             // 推进

            inorder(curr->right);                    // 右
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def treeToDoublyList(self, root):
            if not root: return None
            self.head = None
            self.prev = None
            self._inorder(root)
            # 关环 — self.head 是最小, self.prev 是最大 (中序末位)
            self.head.left = self.prev
            self.prev.right = self.head
            return self.head

        def _inorder(self, curr):
            if not curr: return
            self._inorder(curr.left)
            # 访问 curr
            if self.prev:
                self.prev.right = curr
                curr.left = self.prev
            else:
                self.head = curr
            self.prev = curr
            self._inorder(curr.right)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {Node} root
     * @return {Node}
     */
    var treeToDoublyList = function(root) {
        if (!root) return null;
        // JS 用闭包捕获 head / prev — 比 class 字段简洁
        let head = null, prev = null;
        const inorder = (curr) => {
            if (!curr) return;
            inorder(curr.left);
            if (prev) {
                prev.right = curr;
                curr.left = prev;
            } else {
                head = curr;
            }
            prev = curr;
            inorder(curr.right);
        };
        inorder(root);
        head.left = prev;
        prev.right = head;
        return head;
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问 1 次.
- **Space**: O(h) — 递归栈 (Morris 可到 O(1)).

## 相关题目

- [0094. Binary Tree Inorder Traversal](../../07-binary-tree/0094-binary-tree-inorder-traversal/README.md) — 中序遍历母题
- [0430. Flatten a Multilevel Doubly Linked List](../0430-flatten-a-multilevel-doubly-linked-list/README.md) — DLL 拉平, 同源"双向接指针"
- 0114\. Flatten Binary Tree to Linked List (待补) — 树拉平成单链, 前序 + 返 tail
- 0538\. Convert BST to Greater Tree (待补, 已存) — 反向中序 (右 → 中 → 左)
- 0501\. Find Mode in Binary Search Tree (待补, 已存) — 中序 + prev 检查相邻
- 0530\. Minimum Absolute Difference in BST (待补, 已存) — 中序 + prev 求最小差
- 0173\. Binary Search Tree Iterator (待补) — 迭代版中序, 惰性
- 0708\. Insert into a Sorted Circular Linked List (待补) — 循环双向链表操作

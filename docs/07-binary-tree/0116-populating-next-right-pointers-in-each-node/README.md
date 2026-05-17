# 0116. Populating Next Right Pointers in Each Node / 填充每个节点的下一个右侧节点指针

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, DFS, Linked List · 二叉树, 广度优先搜索, 深度优先搜索, 链表
    - **Link**: [LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a **perfect** binary tree (all leaves at same level, every parent has two children), set each node's `next` pointer to its **right neighbor on the same level**, or `nullptr` if none.

**中文**: 给一棵**完美**二叉树 (所有叶子同层, 每个内部节点都有两个孩子), 把每个节点的 `next` 指向它同层右边的节点, 没有就指 `nullptr`.

## 思路

### 一句话总结

**BFS: [0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 出队时偷看 `queue.front()` 当作 `next`. 完美树版还有递归捷径: `root->left->next = root->right`, `root->right->next = root->next ? root->next->left : nullptr`.**

## Solution

### Variant A — BFS (works for any tree shape)

=== "C++"
    ```cpp
    class Solution {
    public:
        Node* connect(Node* root) {
            queue<Node*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                for (int i = 0; i < size; i++) {
                    Node* cur = que.front(); que.pop();
                    // 本层最后一个的 next 是 nullptr; 否则就是下一个出队的
                    cur->next = (i == size - 1) ? nullptr : que.front();
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
            }
            return root;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def connect(self, root: 'Node | None') -> 'Node | None':
            if not root:
                return None
            q: deque = deque([root])
            while q:
                size = len(q)
                for i in range(size):
                    cur = q.popleft()
                    # q[0] 是 deque 的"队头偷看", 不弹出. 等价 C++ que.front().
                    cur.next = None if i == size - 1 else q[0]
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
            return root
    ```

=== "JavaScript"
    ```javascript
    var connect = function(root) {
        if (!root) return null;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                // queue[0] 是数组下标 0, 偷看队头, 跟 deque[0] / queue.front() 一个意思.
                cur.next = (i === size - 1) ? null : queue[0];
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
        }
        return root;
    };
    ```

### Variant B — recursion exploiting perfect-tree symmetry / 完美树递归

完美树才能用. 思路: `root` 的两个孩子之间靠**父节点**直接连; 跨子树的连接要靠**父节点的 next** 来桥接.

=== "C++"
    ```cpp
    class Solution {
    public:
        Node* connect(Node* root) {
            if (!root) return nullptr;
            if (root->left)  root->left->next  = root->right;
            // 跨子树: 用父节点的 next 找到右邻居的左孩子
            if (root->right) root->right->next = root->next ? root->next->left : nullptr;
            connect(root->left);
            connect(root->right);
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def connect(self, root: 'Node | None') -> 'Node | None':
            if not root:
                return None
            if root.left:
                root.left.next = root.right
            if root.right:
                root.right.next = root.next.left if root.next else None
            self.connect(root.left)
            self.connect(root.right)
            return root
    ```

=== "JavaScript"
    ```javascript
    var connect = function(root) {
        if (!root) return null;
        if (root.left)  root.left.next  = root.right;
        if (root.right) root.right.next = root.next ? root.next.left : null;
        connect(root.left);
        connect(root.right);
        return root;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: BFS O(n) for the queue. Recursion O(h) for the call stack (perfect tree → O(log n)).

## 易错点

- **递归版只在完美树有效**. 通用情况下 (有缺孩子的) 用 [0117](../0117-populating-next-right-pointers-in-each-node-ii/README.md) 的方法.
- **跨子树连接** `root->right->next = root->next ? root->next->left : nullptr` —— 别忘了判 `root->next` 是不是 nullptr (最右那条边上的节点 `next` 都是 null).

## 相关题目

- [0117. Populating Next Right Pointers in Each Node II](../0117-populating-next-right-pointers-in-each-node-ii/README.md) — 通用版 (非完美树)
- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 模板

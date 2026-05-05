# 0117. Populating Next Right Pointers in Each Node II / 填充每个节点的下一个右侧节点指针 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, Linked List · 二叉树, 广度优先搜索, 链表
    - **Link**: [LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Same as [0116](../0116-populating-next-right-pointers-in-each-node/README.md) but the tree is **not necessarily perfect** — nodes can be missing children.

**中文**: 跟 0116 一样, 但树**不一定是完美二叉树** —— 节点可能缺孩子.

## 思路

### 一句话总结

**BFS 方案和 [0116](../0116-populating-next-right-pointers-in-each-node/README.md) 一字不差 —— 因为 BFS 不依赖树的形状. 完美树的递归捷径在这里失效.**

为什么递归捷径失效: 0116 里 `root->right->next = root->next->left` 假设了 `root->next` 一定有左孩子, 通用树里这个左孩子可能不存在, 得继续往 `root->next->right` 找, 甚至 `root->next->next->...`. 想 O(1) 空间递归的话还得用一种"按层链表"的进阶写法 (这里不展开).

## Solution

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
                    if (i < size - 1) cur->next = que.front();   // 本层非末尾 → 偷看队头
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
                    if i < size - 1:
                        cur.next = q[0]   # deque[0] 偷看, 不弹出
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
                if (i < size - 1) cur.next = queue[0];
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
        }
        return root;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n) for the queue. (Constant-space O(1) is possible with a "level-as-linked-list" trick — 进阶, 暂不展开.)

## 易错点

- **`Node` 的 `next` 默认值**: LeetCode 给的初始 `next` 都是 nullptr, 所以本层最后一个节点不需要显式赋值; 上面代码用 `if (i < size - 1)` 跳过最后一个就行.
- **不能用 0116 的递归捷径**: 节点可能缺孩子, "用父节点的 next 找右邻居" 要往右走多步, 写起来比 BFS 还乱.

## 相关题目

- [0116. Populating Next Right Pointers / 完美树版](../0116-populating-next-right-pointers-in-each-node/README.md) — 完美树有递归捷径
- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 模板

# 0111. Minimum Depth of Binary Tree / 二叉树的最小深度

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BFS, DFS, Recursion · 二叉树, 广度优先搜索, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-depth-of-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☑ ☐

## Problem

**EN**: Return the number of nodes on the **shortest** root-to-leaf path. A leaf is a node with no children — a node with one child is *not* a leaf.

**中文**: 返回从根到**最近叶子**的节点数. 叶子 = 左右孩子都没有的节点 (只有一个孩子不算叶子).

## 思路

### 一句话总结

**BFS: 第一个遇到的叶子所在层就是答案 (天然最短路径); 递归: 注意空子树不能直接 `min`, 必须特判.**

## Solution

### Variant A — BFS, short-circuit on first leaf

最自然 —— BFS 每层扩展一圈, 第一个无孩子的节点就是最近叶子.

=== "C++"
    ```cpp
    class Solution {
    public:
        int minDepth(TreeNode* root) {
            int depth = 0;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                depth++;
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    if (!cur->left && !cur->right) return depth;   // 第一个叶子, 立即返回
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
            }
            return 0;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def minDepth(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            q: deque = deque([root])
            depth = 0
            while q:
                depth += 1
                for _ in range(len(q)):
                    cur = q.popleft()
                    if not cur.left and not cur.right:
                        return depth
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
            return 0
    ```

=== "JavaScript"
    ```javascript
    var minDepth = function(root) {
        if (!root) return 0;
        const queue = [root];
        let depth = 0;
        while (queue.length > 0) {
            depth++;
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                if (!cur.left && !cur.right) return depth;
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
        }
        return 0;
    };
    ```

### Variant B — recursion (with the null-child special case)

=== "C++"
    ```cpp
    class Solution {
    public:
        int minDepth(TreeNode* root) {
            if (!root) return 0;
            // 关键: 单边为空时不能取 min(0, 另一边), 否则会算成 1, 错答案.
            // 必须把"单孩子节点"当成"只能往那条非空边走".
            if (!root->left)  return 1 + minDepth(root->right);
            if (!root->right) return 1 + minDepth(root->left);
            return 1 + min(minDepth(root->left), minDepth(root->right));
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minDepth(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            if not root.left:
                return 1 + self.minDepth(root.right)
            if not root.right:
                return 1 + self.minDepth(root.left)
            return 1 + min(self.minDepth(root.left), self.minDepth(root.right))
    ```

=== "JavaScript"
    ```javascript
    var minDepth = function(root) {
        if (!root) return 0;
        if (!root.left)  return 1 + minDepth(root.right);
        if (!root.right) return 1 + minDepth(root.left);
        return 1 + Math.min(minDepth(root.left), minDepth(root.right));
    };
    ```

## Complexity

- **Time**: BFS O(n) 平均更快 (碰到叶子就退); 递归 O(n).
- **Space**: 递归 O(h); BFS O(n).

## 易错点

- **递归直接 `1 + min(left, right)` 是错的**: 当一边是空树时返回 0, 算出来的最小深度会跳过那条不存在的路径, 答案会偏小. 例: 树 `1 → 左空, 右子树有 2`, 正确最短深度是 `2` (1→2 才到叶子), 但 `1 + min(0, 1) = 1` 直接错了.
- **0104 最大深度没这坑**: 因为 `max(0, x) = x`, 空子树天然被忽略. min 不行.

## 相关题目

- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 对称的"最大深度", 没有 null 子树坑
- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 模板

# 0104. Maximum Depth of Binary Tree / 二叉树的最大深度

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BFS, DFS, Recursion · 二叉树, 广度优先搜索, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Return the number of nodes on the longest root-to-leaf path.

**中文**: 返回从根到最远叶子的节点数.

## 思路

### 一句话总结

**BFS: 数完几层就是几; 递归: `1 + max(左深, 右深)`.**

## Solution

### Variant A — recursion / 递归

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxDepth(TreeNode* root) {
            if (!root) return 0;
            return 1 + max(maxDepth(root->left), maxDepth(root->right));
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxDepth(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
    ```

=== "JavaScript"
    ```javascript
    var maxDepth = function(root) {
        if (!root) return 0;
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    };
    ```

### Variant B — BFS counting levels

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxDepth(TreeNode* root) {
            int depth = 0;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                depth++;
            }
            return depth;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def maxDepth(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            q: deque = deque([root])
            depth = 0
            while q:
                for _ in range(len(q)):
                    cur = q.popleft()
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                depth += 1
            return depth
    ```

=== "JavaScript"
    ```javascript
    var maxDepth = function(root) {
        if (!root) return 0;
        const queue = [root];
        let depth = 0;
        while (queue.length > 0) {
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
            depth++;
        }
        return depth;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: 递归 O(h); BFS O(n).

## 相关题目

- [0111. Minimum Depth of Binary Tree](../0111-minimum-depth-of-binary-tree/README.md) — 对称的"最小深度", 但有坑 (空子树不算)
- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 模板
- [0559. Maximum Depth of N-ary Tree / N 叉树的最大深度](../0559-maximum-depth-of-n-ary-tree/README.md) — N 叉版

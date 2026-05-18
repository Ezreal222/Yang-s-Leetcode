# 0199. Binary Tree Right Side View / 二叉树的右视图

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, DFS · 二叉树, 广度优先搜索, 深度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-right-side-view/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Standing on the right of a binary tree, return the values you can see from top to bottom.

**中文**: 从树的右侧看过去, 从上到下能看到的节点值.

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 每层只取**最后一个**节点 (`i == size - 1`).**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> rightSideView(TreeNode* root) {
            vector<int> res;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                int right;
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    if (i == size - 1) right = cur->val;   // 锁定本层最后一个
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                res.push_back(right);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def rightSideView(self, root: 'TreeNode | None') -> list[int]:
            res: list[int] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                size = len(q)
                for i in range(size):
                    cur = q.popleft()
                    if i == size - 1:
                        res.append(cur.val)
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
            return res
    ```

=== "JavaScript"
    ```javascript
    var rightSideView = function(root) {
        const res = [];
        if (!root) return res;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                if (i === size - 1) res.push(cur.val);
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n).

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 模板
- [0515. Find Largest Value in Each Tree Row](../0515-find-largest-value-in-each-tree-row/README.md) — 每层最大, 同款"对每层做归约"

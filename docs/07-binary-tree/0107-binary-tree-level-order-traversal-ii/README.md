# 0107. Binary Tree Level Order Traversal II / 二叉树的层序遍历 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, Queue · 二叉树, 广度优先搜索, 队列
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☑ ☐

## Problem

**EN**: Same as 0102, but return levels **bottom-up** (deepest level first).

**中文**: 跟 0102 一样, 只是结果**自底向上**返回 (最深一层在前).

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 把每层结果 `push_front` 到 deque (或者最后整体 `reverse`).**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> levelOrderBottom(TreeNode* root) {
            deque<vector<int>> res;   // 用 deque 才能 push_front
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                vector<int> vec;
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    vec.push_back(cur->val);
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                res.push_front(vec);   // 唯一的差: 头插
            }
            return vector<vector<int>>(res.begin(), res.end());
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def levelOrderBottom(self, root: 'TreeNode | None') -> list[list[int]]:
            res: list[list[int]] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                level = []
                for _ in range(len(q)):
                    cur = q.popleft()
                    level.append(cur.val)
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                res.append(level)
            # 直接 reverse 比 push_front 简单 (Python 没有 deque 头插不便利的问题, 但
            # list.insert(0, ...) 是 O(n), 还是末尾 append 再 reverse 干净).
            return res[::-1]   # slicing reverse, O(n) 拷贝
    ```

=== "JavaScript"
    ```javascript
    var levelOrderBottom = function(root) {
        const res = [];
        if (!root) return res;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            const level = [];
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                level.push(cur.val);
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
            res.push(level);
        }
        return res.reverse();   // arr.reverse() 原地, 同时返回
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n).

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 正序版, 模板出处

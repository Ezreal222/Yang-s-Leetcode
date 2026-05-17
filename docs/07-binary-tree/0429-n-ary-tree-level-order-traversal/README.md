# 0429. N-ary Tree Level Order Traversal / N 叉树的层序遍历

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, N-ary Tree · N 叉树, 广度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/n-ary-tree-level-order-traversal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Same as 0102 but for N-ary trees (each node has a `children` list of arbitrary size).

**中文**: 跟 0102 一样, 但是 N 叉树 —— 每个节点有一个 `children` 列表.

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 把 `push left/right` 改成**遍历 `children` 全部入队**.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> levelOrder(Node* root) {
            vector<vector<int>> res;
            queue<Node*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                vector<int> vec;
                for (int i = 0; i < size; i++) {
                    Node* cur = que.front(); que.pop();
                    vec.push_back(cur->val);
                    for (Node* child : cur->children) {  // 唯一的差: 多孩子
                        que.push(child);
                    }
                }
                res.push_back(vec);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def levelOrder(self, root: 'Node | None') -> list[list[int]]:
            res: list[list[int]] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                level = []
                for _ in range(len(q)):
                    cur = q.popleft()
                    level.append(cur.val)
                    # deque.extend(iterable): 把整个 iterable 一次性 append 到尾.
                    # 等价 C++ 里 for(child : children) que.push(child).
                    q.extend(cur.children)
                res.append(level)
            return res
    ```

=== "JavaScript"
    ```javascript
    var levelOrder = function(root) {
        const res = [];
        if (!root) return res;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            const level = [];
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                level.push(cur.val);
                // arr.push(...iter) with spread: 一次性把可迭代对象的所有元素 push 进来.
                // 等价 for (child of cur.children) queue.push(child).
                queue.push(...cur.children);
            }
            res.push(level);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n).

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 二叉版
- 0589. N-ary Tree Preorder Traversal (待补)
- 0590. N-ary Tree Postorder Traversal (待补)

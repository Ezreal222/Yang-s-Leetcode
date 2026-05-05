# 0637. Average of Levels in Binary Tree / 二叉树的层平均值

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BFS · 二叉树, 广度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/average-of-levels-in-binary-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Return a list of average values per level.

**中文**: 返回每层节点值的平均数.

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 每层 sum 累加, 出层时 `sum / size` 入结果.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<double> averageOfLevels(TreeNode* root) {
            vector<double> res;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                double levelSum = 0;   // double 防大树整数溢出
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    levelSum += cur->val;
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                res.push_back(levelSum / size);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def averageOfLevels(self, root: 'TreeNode | None') -> list[float]:
            res: list[float] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                size = len(q)
                level_sum = 0
                for _ in range(size):
                    cur = q.popleft()
                    level_sum += cur.val
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                res.append(level_sum / size)   # Python 3 / 是真除, 不会整除截断
            return res
    ```

=== "JavaScript"
    ```javascript
    var averageOfLevels = function(root) {
        const res = [];
        if (!root) return res;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            let levelSum = 0;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                levelSum += cur.val;
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
            res.push(levelSum / size);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n).

## 易错点

- **C++ 用 `double` 累加**: 极端大树时层和可能超 `int`. `double` 精度也够这题.

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 模板
- [0515. Find Largest Value in Each Tree Row](../0515-find-largest-value-in-each-tree-row/README.md) — 每层 max, 同款 reduce

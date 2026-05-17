# 0515. Find Largest Value in Each Tree Row / 在每个树行中找最大值

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS · 二叉树, 广度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/find-largest-value-in-each-tree-row/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Return the largest value in each row of a binary tree.

**中文**: 返回每一层的最大值.

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板, 每层维护一个 `levelMax`, 出层时 push 进结果.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> largestValues(TreeNode* root) {
            vector<int> res;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                int levelMax = INT_MIN;   // 节点值可能为负, 用 INT_MIN 作哨兵
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    levelMax = max(levelMax, cur->val);
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                res.push_back(levelMax);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def largestValues(self, root: 'TreeNode | None') -> list[int]:
            res: list[int] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                # float('-inf'): Python 浮点的负无穷哨兵, 比写一个具体大负数干净.
                #   C++ 等价: INT_MIN (但这里用浮点仅作哨兵, 比较不影响).
                level_max = float('-inf')
                for _ in range(len(q)):
                    cur = q.popleft()
                    if cur.val > level_max:
                        level_max = cur.val
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                res.append(level_max)
            return res
    ```

=== "JavaScript"
    ```javascript
    var largestValues = function(root) {
        const res = [];
        if (!root) return res;
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;
            // -Infinity: JS 内置的负无穷常量, 同 Number.NEGATIVE_INFINITY.
            // 任何数都比它大, 适合做 max-reduce 的初始值.
            let levelMax = -Infinity;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                if (cur.val > levelMax) levelMax = cur.val;
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
            res.push(levelMax);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(n).

## 易错点

- **`levelMax` 初始值**: 节点值可能是负数, 别用 `0` 当初始值. C++ 写 `INT_MIN`, Python `float('-inf')`, JS `-Infinity`.

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 模板
- [0637. Average of Levels](../0637-average-of-levels-in-binary-tree/README.md) — 同款"对每层归约", reduce 换成 avg
- [0199. Binary Tree Right Side View](../0199-binary-tree-right-side-view/README.md) — 每层"取最后一个"

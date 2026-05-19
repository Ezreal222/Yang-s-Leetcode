# 0513. Find Bottom Left Tree Value / 找树左下角的值

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, DFS · 二叉树, 广度优先搜索, 深度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/find-bottom-left-tree-value/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Return the leftmost value in the **last** (deepest) row of a binary tree.

**中文**: 返回最深一层最左边节点的值.

## 思路

### 一句话总结

**[0102](../0102-binary-tree-level-order-traversal/README.md) BFS 模板, 每层只记下第一个节点 (`i == 0`); 循环结束时, `res` 自然就是最深一层的第一个 —— 也就是左下角.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findBottomLeftValue(TreeNode* root) {
            int res = 0;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    if (i == 0) res = cur->val;   // 锁住本层第一个; 最后一次写就是答案
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def findBottomLeftValue(self, root: 'TreeNode') -> int:
            q: deque = deque([root])
            res = root.val
            while q:
                for i in range(len(q)):
                    cur = q.popleft()
                    if i == 0:
                        res = cur.val
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var findBottomLeftValue = function(root) {
        const queue = [root];
        let res = root.val;
        while (queue.length > 0) {
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                if (i === 0) res = cur.val;
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

## 易错点

- **题目保证 root 非空**: 所以 Python/JS 直接 `res = root.val` 作初值就行; C++ 用 `int res = 0` 因为入口的 `if (root)` 保证不会进 while 跑空表.
- **小优化**: BFS 也可以**从右向左**入队 (先 push right 再 push left), 这样**最后一个**出队的就是左下角, 不需要 `i == 0` 判定. 写法更紧凑, 但跟模板不一致, 看个人.
- **DFS 也行**: 维护"目前最大深度"和"那一层第一个值"两个变量, 前序遍历, 第一次到达更深层时记录. 思路对但代码更长, 没必要.

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 模板出处
- [0199. Binary Tree Right Side View](../0199-binary-tree-right-side-view/README.md) — 对称版: 每层最后一个 (`i == size - 1`)
- [0404. Sum of Left Leaves](../0404-sum-of-left-leaves/README.md) — 同款"在父节点视角找左侧"的思路
- 0515. Find Largest Value in Each Tree Row — 同模板, 改 reduce

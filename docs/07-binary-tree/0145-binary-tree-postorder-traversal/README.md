# 0145. Binary Tree Postorder Traversal / 二叉树的后序遍历

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Stack · 二叉树, 深度优先搜索, 栈
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-postorder-traversal/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem / 题意

**EN**: Return the **postorder** traversal (left → right → root) of a binary tree.

**中文**: 返回二叉树的**后序**遍历 (左 → 右 → 根).

## Approach / 思路

### 核心思想 / Core idea

跟 [0144 preorder](../0144-binary-tree-preorder-traversal/README.md) 一模一样, 只是把 "访问 root" 这行从最前**挪到最后** —— 先递归左、再递归右、最后 push 当前节点.

### 关键洞察 / Key insights

**迭代版的小技巧**: postorder = "**根 → 右 → 左**" 然后**整体 reverse**. 因为根→右→左其实是 preorder 的镜像, 拿 0144 的迭代代码改两行 (左右 push 顺序对调) 跑出来再反转就是答案. 比直接写 postorder 迭代简单太多.

C++ helper 的 `vector<int>&` 引用传递坑跟 0144 一样, 见那边讨论.

### 一句话总结 / One-liner

**Postorder = visit 移到最后. 迭代偷懒: preorder 镜像 + reverse.**

## Solution / 题解

=== "C++"
    ```cpp
    class Solution {
    public:
        void traversal(TreeNode* cur, vector<int>& vec) {
            if (cur == nullptr) return;
            traversal(cur->left,  vec);
            traversal(cur->right, vec);
            vec.push_back(cur->val);   // root 移到最后
        }
        vector<int> postorderTraversal(TreeNode* root) {
            vector<int> res;
            traversal(root, res);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def postorderTraversal(self, root: 'TreeNode | None') -> list[int]:
            res: list[int] = []
            def dfs(node):
                if not node:
                    return
                dfs(node.left)
                dfs(node.right)
                res.append(node.val)   # root 最后
            dfs(root)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number[]}
     */
    var postorderTraversal = function(root) {
        const res = [];
        const dfs = (node) => {
            if (!node) return;
            dfs(node.left);
            dfs(node.right);
            res.push(node.val);   // root 最后
        };
        dfs(root);
        return res;
    };
    ```

## Complexity / 复杂度

- **Time**: O(n).
- **Space**: O(h) for recursion (h = tree height; worst O(n) skewed, avg O(log n) balanced) + O(n) output.

## Pitfalls / 易错点

- C++ helper 必须 `vector<int>&` 不要值传 —— 同 0144.
- 别误以为 "把递归代码 reverse 一下就是 postorder 迭代" —— 那只是迭代版的偷懒做法, 递归本身改 visit 位置就行.

## Related / 相关题目

- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 完整讨论 (`&` 引用、迭代模板) 都在这里
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 三兄弟之中位
- 0102. Binary Tree Level Order Traversal (待补) — BFS

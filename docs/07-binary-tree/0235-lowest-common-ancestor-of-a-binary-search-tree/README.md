# 0235. Lowest Common Ancestor of a Binary Search Tree / 二叉搜索树的最近公共祖先

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion, LCA · 二叉树, 二叉搜索树, 深度优先搜索, 递归, 最近公共祖先
    - **Link**: [LeetCode](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Find the LCA of two nodes `p` and `q` in a **BST** (no parent pointers).

**中文**: 在 **BST** 里找两个节点 `p` 和 `q` 的最近公共祖先 (没有父指针).

## 思路

### 一句话总结

**[0530 口诀](../0530-minimum-absolute-difference-in-bst/README.md#key-insights): 操作 BST 用二分. 当前节点 `root.val`: 比 p、q 都大 → 答案在左; 都小 → 答案在右; 否则 (一个在两侧, 或者撞上 p/q 本身) → 当前就是 LCA.** O(h) 一条路径走到底, 跟 [0236 普通树版](../0236-lowest-common-ancestor-of-a-binary-tree/README.md) 必须两边都跑形成鲜明对比.

为什么"一个在两侧"就是 LCA: 如果 `p.val < root.val < q.val` (或反过来), p 和 q 必然分居左右子树, root 就是第一个把它们分开的祖先.

## Solution

### Variant A — recursion

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
            // root 比两者都大 → 答案在左子树
            if (root->val > p->val && root->val > q->val)
                return lowestCommonAncestor(root->left, p, q);
            // root 比两者都小 → 答案在右子树
            if (root->val < p->val && root->val < q->val)
                return lowestCommonAncestor(root->right, p, q);
            // 一个在两侧, 或撞上 p/q 自己 → 当前就是 LCA
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
            if root.val > p.val and root.val > q.val:
                return self.lowestCommonAncestor(root.left, p, q)
            if root.val < p.val and root.val < q.val:
                return self.lowestCommonAncestor(root.right, p, q)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {TreeNode} p
     * @param {TreeNode} q
     * @return {TreeNode}
     */
    var lowestCommonAncestor = function(root, p, q) {
        if (root.val > p.val && root.val > q.val) return lowestCommonAncestor(root.left, p, q);
        if (root.val < p.val && root.val < q.val) return lowestCommonAncestor(root.right, p, q);
        return root;
    };
    ```

### Variant B — iterative (no stack needed)

BST 搜索是单向的, 直接 while 循环就够, 不用栈.

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
            while (root) {
                if (root->val > p->val && root->val > q->val)      root = root->left;
                else if (root->val < p->val && root->val < q->val) root = root->right;
                else return root;
            }
            return nullptr;   // 题目保证 p、q 都存在, 实际不会走到
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
            while root:
                if root.val > p.val and root.val > q.val:
                    root = root.left
                elif root.val < p.val and root.val < q.val:
                    root = root.right
                else:
                    return root
            return None
    ```

=== "JavaScript"
    ```javascript
    var lowestCommonAncestor = function(root, p, q) {
        while (root) {
            if (root.val > p.val && root.val > q.val)      root = root.left;
            else if (root.val < p.val && root.val < q.val) root = root.right;
            else return root;
        }
        return null;
    };
    ```

## Complexity

- **Time**: O(h) — 单向走一条路径; 平衡 BST 是 O(log n), 退化是 O(n).
- **Space**: 递归 O(h); 迭代 O(1).

## 易错点

- **`>` 和 `<` 的语义别写反**: `root > p && root > q` 意味着 p、q 都比 root 小, 都**在左**子树. 写反就找去错的方向, 直接 stack overflow / 永远找不到.
- **不需要短路 OR 两路都跑**: 跟 [0236 普通树版](../0236-lowest-common-ancestor-of-a-binary-tree/README.md) 区别. BST 因为有序, 每次只走一条路径; 普通树没这奢侈, 必须左右都递归再合并.
- **撞上 p 或 q 自己也算 LCA**: 题目里 `root` 等于 `p` 或 `q` 时, 第三个 return 自动 return root. 不需要单独 base case (Yang's code 就是这么写的).
- **空 root 不需要保护**: 题目保证 p、q 都存在, 所以从根开始走永远会撞到第一个有效情况, 不会走到 null. 工程代码可以加 `if (!root) return nullptr;` 防御但不必须.
- **BST LCA vs 普通 LCA 的复杂度对比**: 普通版 O(n) 因为要走全树; BST 版 O(h), 平衡时 O(log n). 这是 BST 性质值钱的地方.

## 相关题目

- [0236. Lowest Common Ancestor of a Binary Tree](../0236-lowest-common-ancestor-of-a-binary-tree/README.md) — 普通树版, 必须两边都跑 + 后序合并
- [0700. Search in a Binary Search Tree](../0700-search-in-a-binary-search-tree/README.md) — 同款"BST 单向走"骨架
- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — BST 性质入门讨论
- 0701. Insert into a BST (待补) — 同款单向走找空位
- 0450. Delete Node in a BST (待补) — 同款单向走找节点

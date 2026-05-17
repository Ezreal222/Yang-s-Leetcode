# 0617. Merge Two Binary Trees / 合并二叉树

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, BFS, Recursion · 二叉树, 深度优先搜索, 广度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/merge-two-binary-trees/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given two binary trees, return a merged tree: at every position, the merged node's value = sum of the two corresponding nodes' values; if only one tree has a node at that position, the merged tree takes that node directly.

**中文**: 给两棵二叉树, 返回合并后的树: 对应位置都有节点 → 值相加; 只有一边有节点 → 直接取那边.

## 思路

### 一句话总结

**双递归同步走两棵树, 对应位置都在 → 新节点 = 两值相加 + 左右子树各自递归合并; 一边为空 → 直接返回另一边 (整棵子树).**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
            if (!root1) return root2;        // 一边空, 直接接另一边整棵子树
            if (!root2) return root1;
            TreeNode* cur = new TreeNode(root1->val + root2->val);
            cur->left  = mergeTrees(root1->left,  root2->left);
            cur->right = mergeTrees(root1->right, root2->right);
            return cur;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def mergeTrees(self, root1: 'TreeNode | None', root2: 'TreeNode | None') -> 'TreeNode | None':
            if not root1: return root2
            if not root2: return root1
            cur = TreeNode(root1.val + root2.val)
            cur.left  = self.mergeTrees(root1.left,  root2.left)
            cur.right = self.mergeTrees(root1.right, root2.right)
            return cur
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root1
     * @param {TreeNode} root2
     * @return {TreeNode}
     */
    var mergeTrees = function(root1, root2) {
        if (!root1) return root2;
        if (!root2) return root1;
        const cur = new TreeNode(root1.val + root2.val);
        cur.left  = mergeTrees(root1.left,  root2.left);
        cur.right = mergeTrees(root1.right, root2.right);
        return cur;
    };
    ```

## Complexity

- **Time**: O(min(n₁, n₂)) — 只在两边都有节点时才递归, 一边空就直接挂过去.
- **Space**: O(min(h₁, h₂)) recursion.

## 易错点

- **`if (!root1 && !root2) return nullptr;` 是死代码**: 你原版加了这一行 (放在前两个 if 之后), 但**永远走不到** —— 如果 `root1` 是 null, 第一个 `if (!root1) return root2` 已经返回 `root2` (恰好也是 null 时返回 null, 正确). 删掉这行不影响正确性, 减少噪音.
- **复用 root1 还是新建节点**: 上面的实现**新建**节点, 不破坏输入. 也可以**原地合并** —— 把 root2 的值加到 root1 上, 复用 root1 整棵树, 省空间但破坏输入. 看 spec 允不允许.
- **一边空时直接 return 另一边**: 不要继续递归 —— 那一边整棵子树都直接挂过来即可. 写成 `mergeTrees(null, root2.left)` 会浪费一层调用, 虽然结果也对.
- **BFS 版本也可以**: 用一个队列同步出队两棵树的对应节点, 处理 `(n1, n2)` 对. 思路一样, 写起来比递归长.

## 相关题目

- [0101. Symmetric Tree](../0101-symmetric-tree/README.md) — 同款"双递归同步两棵树"
- [0226. Invert Binary Tree](../0226-invert-binary-tree/README.md) — 同款递归骨架
- [0105. Construct Binary Tree from Preorder and Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 同款双递归 (那边是两个数组)
- [0106. Construct Binary Tree from Inorder and Postorder](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 同上
- 0100. Same Tree (待补) — 同款双递归, 但是判等不是合并

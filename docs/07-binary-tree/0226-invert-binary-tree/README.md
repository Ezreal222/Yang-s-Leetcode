# 0226. Invert Binary Tree / 翻转二叉树

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, BFS, Recursion · 二叉树, 深度优先搜索, 广度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/invert-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Mirror a binary tree — at every node, swap its left and right subtrees.

**中文**: 把二叉树镜像翻转 —— 每个节点的左右子树互换.

## 思路

### 一句话总结

**每个节点 swap 左右孩子, 然后递归处理两边. 前序 (swap 再递归) 或后序 (递归再 swap) 都行, 中序不行.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* invertTree(TreeNode* root) {
            if (!root) return root;
            swap(root->left, root->right);
            invertTree(root->left);
            invertTree(root->right);
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def invertTree(self, root: 'TreeNode | None') -> 'TreeNode | None':
            if not root:
                return root
            # Python tuple swap: 右边先打包成临时元组再解包赋给左边. 一行原子完成,
            # 不用临时变量. 这是 Python 最干净的 swap 写法.
            #   C++ 等价: std::swap(root->left, root->right);
            root.left, root.right = root.right, root.left
            self.invertTree(root.left)
            self.invertTree(root.right)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {TreeNode}
     */
    var invertTree = function(root) {
        if (!root) return root;
        // ES6 array destructuring swap: 跟 Python 思路一样, 把右边数组解构给左边.
        // 不用临时变量, 一行搞定.
        //   C++ 等价: std::swap(root->left, root->right);
        [root.left, root.right] = [root.right, root.left];
        invertTree(root.left);
        invertTree(root.right);
        return root;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(h) recursion stack (h = height; worst O(n) skewed, avg O(log n) balanced).

## 易错点

- **中序不行**: 写成 `recurse(left); swap; recurse(right)` 会**先**递归原始左, swap 之后**再**递归"swap 后的右"(其实就是原始左!) → 同一棵子树被处理两次, 另一棵被忽略, 结果错乱. 前序和后序都没这问题.
- **空树**: 别忘了 `if (!root) return root;`. 否则 `root->left` 解空指针崩.

## 相关题目

- [0101. Symmetric Tree / 对称二叉树](../0101-symmetric-tree/README.md) — "翻转后是否相等" 的判定问题
- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 同款递归骨架
- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 这里的 swap+递归 就是前序的"访问→左→右"

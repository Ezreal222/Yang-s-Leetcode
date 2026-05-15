# 0700. Search in a Binary Search Tree / 二叉搜索树中的搜索

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/search-in-a-binary-search-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: In a BST, find and return the subtree rooted at the node whose value equals `val` (or null if not found).

**中文**: 在 BST 中找到值等于 `val` 的节点, 返回以它为根的子树 (没有就返回 null).

## 思路

### Core idea

**BST 性质**: 对任意节点, **左子树所有值 < 当前 < 右子树所有值**. 所以查找时:

- `cur.val == val` → 找到, 返回.
- `cur.val > val` → 答案只可能在**左**子树.
- `cur.val < val` → 答案只可能在**右**子树.

整个搜索是**单向**走的 —— 从来不需要回头, 所以迭代比递归还干净 (不用栈).

### 一句话总结

**BST 搜索 = 单向走: `val < cur` 往左, `val > cur` 往右, 等就返回. 不用栈, 迭代版甚至比递归版还短.**

## Solution

### Variant A — recursion

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* searchBST(TreeNode* root, int val) {
            if (!root) return nullptr;
            if (root->val == val) return root;
            return root->val > val
                ? searchBST(root->left,  val)
                : searchBST(root->right, val);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def searchBST(self, root: 'TreeNode | None', val: int) -> 'TreeNode | None':
            if not root or root.val == val:
                return root
            return (self.searchBST(root.left,  val) if root.val > val
                    else self.searchBST(root.right, val))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} val
     * @return {TreeNode}
     */
    var searchBST = function(root, val) {
        if (!root || root.val === val) return root;
        return root.val > val ? searchBST(root.left, val) : searchBST(root.right, val);
    };
    ```

### Variant B — iterative (no stack needed)

BST 搜索是**单向**的, 所以不像一般二叉树迭代要用栈 —— 一个 while 循环就够.

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* searchBST(TreeNode* root, int val) {
            while (root && root->val != val) {
                root = root->val > val ? root->left : root->right;
            }
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def searchBST(self, root: 'TreeNode | None', val: int) -> 'TreeNode | None':
            while root and root.val != val:
                root = root.left if root.val > val else root.right
            return root
    ```

=== "JavaScript"
    ```javascript
    var searchBST = function(root, val) {
        while (root && root.val !== val) {
            root = root.val > val ? root.left : root.right;
        }
        return root;
    };
    ```

## Complexity

- **Time**: O(h) — h = tree height. 平衡 BST 是 O(log n), 退化成链表是 O(n).
- **Space**: 递归 O(h), 迭代 O(1).

## 易错点

- **比较方向别写反**: `cur.val > val` 时往**左**走 (因为 `val` 比当前小). 写反了搜索会一直跑错方向, 直接返回 null.
- **BST 搜索不需要栈**: 跟普通二叉树搜索 (要 DFS/BFS) 不一样, BST 因为有序属性, 每步唯一确定走哪边. 迭代版 O(1) 空间是免费午餐, 优先选.
- **`return root` 在空和找到两种情况都对**: Python/JS 版本利用了这一点, `root is None` 时 `root` 就是 `None`, 直接 return 就行, 不用单独写 `return None`.
- **后续 BST 题都靠这个性质**: 0098 (validate BST), 0230 (kth smallest), 0501 (mode), 0701 (insert), 0450 (delete) —— 所有 BST 题的入口都是这个"左 < 当前 < 右"的不变量. 这题就是 BST 系列的 hello world.

## 相关题目

- [0098. Validate Binary Search Tree / 验证二叉搜索树](../0098-validate-binary-search-tree/README.md) — 用 BST 性质判合法 (inorder 升序 / 区间约束两种思路)
- 0230. Kth Smallest Element in a BST (待补) — inorder 第 k 个
- 0501. Find Mode in BST (待补) — inorder 上找众数
- 0701. Insert into a BST (待补) — 同款单向走, 找空位插入
- 0450. Delete Node in a BST (待补) — 同款单向走, 找节点删除 (重组子树)
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — BST inorder 出来就是升序, 后续题反复用

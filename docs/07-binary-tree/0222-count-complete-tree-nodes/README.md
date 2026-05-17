# 0222. Count Complete Tree Nodes / 完全二叉树的节点个数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Binary Search · 二叉树, 深度优先搜索, 二分查找
    - **Link**: [LeetCode](https://leetcode.com/problems/count-complete-tree-nodes/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Count the nodes of a **complete** binary tree (all levels filled except possibly the last, last level filled left-to-right).

**中文**: 数一棵**完全二叉树**的节点个数 (除了最后一层都填满, 最后一层从左往右连续).

## 思路

### 一句话总结

**通用递归 `1 + count(L) + count(R)` 是 O(n), 直接能过. 利用"完全"性质能做到 O(log²n): 每次比一次 leftmost vs rightmost 深度, 相等说明是完美子树, 直接 `2^h - 1`; 不等才递归.**

## Solution

### Variant A — naive recursion / 通用递归 O(n)

不利用任何"完全"特性, 当普通二叉树数. 题目数据规模下能过, 也最好写.

=== "C++"
    ```cpp
    class Solution {
    public:
        int countNodes(TreeNode* root) {
            if (!root) return 0;
            return 1 + countNodes(root->left) + countNodes(root->right);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def countNodes(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            return 1 + self.countNodes(root.left) + self.countNodes(root.right)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var countNodes = function(root) {
        if (!root) return 0;
        return 1 + countNodes(root.left) + countNodes(root.right);
    };
    ```

### Variant B — exploit completeness, O(log² n) / 利用完全性质

**关键观察**: 完全二叉树的每个子树要么本身**完美** (perfect, 满层), 要么**只有一边孩子有空位**. 所以从一个节点出发, 一路走左边量出 `leftDepth`, 一路走右边量出 `rightDepth`:

- `leftDepth == rightDepth` → 这棵子树是完美的, 节点数直接 `2^(depth+1) - 1`, 不用递归.
- 否则 → 两边都还得递归数, 但只有**一边**会继续触发递归 (另一边必然是完美的, O(log n) 就返回).

每层最多走两条 O(log n) 的路径, 共 O(log n) 层, 所以总时间 **O(log² n)**.

=== "C++"
    ```cpp
    class Solution {
    public:
        int countNodes(TreeNode* root) {
            if (!root) return 0;
            // 量左、右子树的"最左/最右深度" —— 都是 O(log n)
            TreeNode* l = root, *r = root;
            int lh = 0, rh = 0;
            while (l->left)  { l = l->left;  lh++; }
            while (r->right) { r = r->right; rh++; }
            // 完美子树: 节点数 = 2^(高度+1) - 1
            if (lh == rh) return (1 << (lh + 1)) - 1;
            // 否则递归 —— 完全性保证至少一边是完美的, 只走 O(log n) 就返回
            return 1 + countNodes(root->left) + countNodes(root->right);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def countNodes(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            l, r = root, root
            lh = rh = 0
            while l.left:
                l = l.left
                lh += 1
            while r.right:
                r = r.right
                rh += 1
            if lh == rh:
                # bit shift: 1 << k 等价 2^k. 这里 (1 << (lh+1)) - 1 = 2^(lh+1) - 1.
                #   C++ 等价: (1 << (lh + 1)) - 1
                return (1 << (lh + 1)) - 1
            return 1 + self.countNodes(root.left) + self.countNodes(root.right)
    ```

=== "JavaScript"
    ```javascript
    var countNodes = function(root) {
        if (!root) return 0;
        let l = root, r = root;
        let lh = 0, rh = 0;
        while (l.left)  { l = l.left;  lh++; }
        while (r.right) { r = r.right; rh++; }
        if (lh === rh) return (1 << (lh + 1)) - 1;
        return 1 + countNodes(root.left) + countNodes(root.right);
    };
    ```

## Complexity

| | Time | Space |
|---|---|---|
| A. naive recursion | O(n) | O(h) |
| B. exploit completeness | **O(log² n)** | O(h) |

A 写起来短, B 是这道题的"正解". 面试时给 B, 但要能背出来.

## 易错点

- **`(1 << (lh + 1)) - 1`**: 高度 `h` 的完美二叉树 (root 是高度 0 那一层) 节点数是 `2^(h+1) - 1`. 这里 `lh` 是从 root 走 leftmost 经过的**边数** —— 树高 = `lh`, 节点数 = `2^(lh+1) - 1`. 别少写 +1.
- **量深度别忘起点**: `l = root` 和 `r = root`, 然后 `while (l->left)` 才走 —— 直接 `l = root->left` 也行但要相应调整高度计算.
- **JS 的 `<<` 在节点数 ≥ 2³¹ 时会溢出**: 这题约束 n ≤ 5×10⁴, 完全没问题; 但要意识到 JS 位运算是 32-bit signed, 大数据要换 `Math.pow(2, ...)` 或 `BigInt`.

## 相关题目

- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 同款"沿一条路径数层数", 但比这里简单 (只走 leftmost 不必比 rightmost)
- [0226. Invert Binary Tree](../0226-invert-binary-tree/README.md) — 同款双递归骨架
- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 同款"`1 + 左 + 右`"递归形式

# 0404. Sum of Left Leaves / 左叶子之和

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/sum-of-left-leaves/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Sum the values of all **left leaves** in the tree (a left leaf = a node that is its parent's left child AND has no children of its own).

**中文**: 求所有**左叶子**的和 (左叶子 = 是父节点的左孩子, 同时自己又是叶子).

## 思路

### 一句话总结

**叶子自己不知道自己是不是"左叶子" —— 只有父节点知道. 所以判定要在**父节点**做: 站在 `root`, 如果 `root.left` 存在且是叶子, 它就是左叶子, 累加它的值; 然后递归两边.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int sumOfLeftLeaves(TreeNode* root) {
            if (!root) return 0;
            // 在父节点视角检查: 左孩子存在且左孩子没有任何孩子 → 它是左叶子
            int val = (root->left && !root->left->left && !root->left->right)
                          ? root->left->val
                          : 0;
            return val + sumOfLeftLeaves(root->left) + sumOfLeftLeaves(root->right);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sumOfLeftLeaves(self, root: 'TreeNode | None') -> int:
            if not root:
                return 0
            val = 0
            if root.left and not root.left.left and not root.left.right:
                val = root.left.val
            return val + self.sumOfLeftLeaves(root.left) + self.sumOfLeftLeaves(root.right)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var sumOfLeftLeaves = function(root) {
        if (!root) return 0;
        let val = 0;
        if (root.left && !root.left.left && !root.left.right) {
            val = root.left.val;
        }
        return val + sumOfLeftLeaves(root.left) + sumOfLeftLeaves(root.right);
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion stack.

## 易错点

- **不能在叶子节点本身判断**: 一个叶子拿到自己的指针, 看不到父节点, 不知道自己是 left 还是 right. 必须从父节点视角看 "我的左孩子是不是叶子" —— 这是这题最容易绕的地方.
- **别漏 `root->left` 非空检查**: `root->left->left` 解空会崩. 三元里靠**短路**保证 (左孩子是 nullptr 时后两个条件不会执行).
- **递归别忘了走两边**: 即使当前 `root` 没贡献左叶子, 它的左/右子树里还可能有 —— 必须递归 `root->left` AND `root->right`.
- **迭代版**: 用栈或队列做层序/DFS 也行, 模板套 0102 BFS 改一下: 出队时检查 `if cur.left and is_leaf(cur.left)` 累加. 思路一样, 写起来比递归长.

## 相关题目

- [0257. Binary Tree Paths](../0257-binary-tree-paths/README.md) — 同款"递归收集叶子相关信息"
- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 同款"自上而下递归 + 子树结果合并"骨架
- [0513. Find Bottom Left Tree Value / 找树左下角的值](../0513-find-bottom-left-tree-value/README.md) — "最底层最左节点", BFS 套用 0102 模板
- [0112. Path Sum / 路径总和](../0112-path-sum/README.md) — 同款叶子判定, 但要同时累加路径和

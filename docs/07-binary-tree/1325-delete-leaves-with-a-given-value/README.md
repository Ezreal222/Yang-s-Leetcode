# 1325. Delete Leaves With a Given Value / 删除给定值的叶子节点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/delete-leaves-with-a-given-value/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Delete every leaf whose value equals `target`. Repeat until no such leaves remain (deleting a leaf can expose a new one whose value is also `target`).

**中文**: 反复删除值等于 `target` 的叶子节点 (一次删除可能让新的"等于 target 的叶子" 暴露出来), 直到没有这种叶子为止.

## 思路

### 一句话总结

**[0814 Pruning](../0814-binary-tree-pruning/README.md) 的同款代码, 把 `val == 0` 换成 `val == target` —— 后序递归, 先剪两边孩子, 再判"我现在是不是 target 叶子". 级联删除靠递归自然完成: 内部 target 子树先把孩子剪光, 它就降级成新的 target 叶子, 上一层再剪掉它.**

### Key Insight: 为什么递归一次就够 / Why one pass suffices

题面说"反复删除", 听起来像要循环跑多遍. 其实**后序一次就够**:

- 进入子树, 先剪左; 再剪右; 然后判自己.
- 子树里所有"原本不是叶子但因为孩子被剪而变成 target 叶子"的节点, 都会在**回溯的时候**被它们的父亲调用判到, 一次干掉.
- 整个删除链从最深处往上**冒泡式自动级联**.

跟 [0814](../0814-binary-tree-pruning/README.md) 完全一样的逻辑, 也是 [0669 Trim BST](../0669-trim-a-binary-search-tree/README.md) Key Insight #2 里的"判删取决于子树状态 → 后序"分类.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* removeLeafNodes(TreeNode* root, int target) {
            if (!root) return nullptr;
            // 后序: 先递归剪两个孩子
            root->left  = removeLeafNodes(root->left,  target);
            root->right = removeLeafNodes(root->right, target);
            // 此刻我可能是: 原本叶子 + val==target, 或 内部节点孩子被剪光 + val==target
            if (!root->left && !root->right && root->val == target) return nullptr;
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def removeLeafNodes(self, root: 'TreeNode | None', target: int) -> 'TreeNode | None':
            if not root:
                return None
            root.left  = self.removeLeafNodes(root.left,  target)
            root.right = self.removeLeafNodes(root.right, target)
            if not root.left and not root.right and root.val == target:
                return None
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} target
     * @return {TreeNode}
     */
    var removeLeafNodes = function(root, target) {
        if (!root) return null;
        root.left  = removeLeafNodes(root.left,  target);
        root.right = removeLeafNodes(root.right, target);
        if (!root.left && !root.right && root.val === target) return null;
        return root;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次, 级联删除全在一次后序里完成.
- **Space**: O(h) recursion.

## 易错点

- **"反复删除" 不需要外层循环**: 后序递归一次就把所有级联干完. 写两层 (外层 while, 内层 dfs) 是冗余 + 慢, 而且不优雅.
- **三个 AND 都要写齐**: `!root->left && !root->right && root->val == target` —— 三者缺一不可. 漏 val 检查会把所有叶子都删 (包含非 target 的); 漏孩子 null 检查会把"我是 target 但孩子里还有非 target"的内部节点也删.
- **整棵树都能被删光**: 顶层 root 也可能最后变成 target 叶子被剪. 返回 null 是合法的, 调用方要接住.
- **同 0814 比较**: 0814 是删"全 0 子树", 这里是删"所有 target 叶子". 但因为级联自动从孩子被剪空开始往上传, 实质等价: 0814 删的是"全 0 的子树", 这题删的是"全 target 的子树" —— 同一份代码, 不同变量名.

## 相关题目

- [0814. Binary Tree Pruning](../0814-binary-tree-pruning/README.md) — 完全同款代码, 只是 target = 0 hardcoded
- [0669. Trim a Binary Search Tree](../0669-trim-a-binary-search-tree/README.md) — 树删除模板, 前序版
- [0450. Delete Node in a BST](../0450-delete-node-in-a-bst/README.md) — 删单点要重组子树
- [1110. Delete Nodes And Return Forest](../1110-delete-nodes-and-return-forest/README.md) — 删除模板 + 收集森林根
- [0112. Path Sum](../0112-path-sum/README.md) — 同款后序递归 + 用孩子返回判自己

# 0814. Binary Tree Pruning / 二叉树剪枝

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-pruning/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given a binary tree where each node's `val` is 0 or 1, remove every subtree that contains **only 0s**. Return the pruned tree.

**中文**: 给一棵节点值为 0/1 的二叉树, 把**所有全是 0 的子树**都剪掉, 返回剪枝后的树.

## 思路

### Core idea

跟 [0669 Trim BST](../0669-trim-a-binary-search-tree/README.md) 用同一个**树删除通用模板**, 但这题"删不删"取决于**整棵子树都是 0 吗?** —— 这是孩子的属性, 必须**先递归再判** → 后序.

### Key Insights

**`pre vs post` 取决于判删条件用谁的信息 / Pre or post by predicate's source**

跟 [0669 Pitfalls](../0669-trim-a-binary-search-tree/README.md#key-insights) 的对比明确化:

| 题目 | 判删条件 | 递归阶段 |
|---|---|---|
| **0669 Trim BST** | `root.val < low / > high` —— 只看自己 | **前序** (判完再递归) |
| **0814 Pruning** (本题) | "整个子树是不是全 0" —— 看孩子 | **后序** (先递归再判) |

后序的妙处: 递归先把全 0 的子子树剪成 null. 然后回到本节点时, `root.left == null && root.right == null` 等价于"我原来这棵子树里, 除了我自己 (现在还没判) 都已经剪光了". 这时如果 `root.val == 0`, 我整棵子树确实就是全 0, 返 null 完成自我清除. 否则保留 (我自己是 1, 或者孩子里有 1 没被剪).

**关键不变量**: 递归返回后, `root.left` / `root.right` 要么是 null (已剪), 要么是"含至少一个 1 的合法子树". 凭这两个状态判自己就够了.

### 一句话总结

**后序 DFS + 树删除模板: 先递归剪两个孩子, 然后用"孩子都剪光了且我是 0"判删自己. 对比 0669: 那题判删用自己, 是前序; 这题判删用孩子, 是后序.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* pruneTree(TreeNode* root) {
            if (!root) return nullptr;
            // 后序: 先递归两个孩子, 让它们各自决定是否剪光
            root->left  = pruneTree(root->left);
            root->right = pruneTree(root->right);
            // 现在 left/right 要么 null 要么"含 1 的子树".
            // 自己是 0 且两边都被剪光 → 我这棵子树全 0, 删掉
            if (!root->left && !root->right && root->val == 0) return nullptr;
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def pruneTree(self, root: 'TreeNode | None') -> 'TreeNode | None':
            if not root:
                return None
            root.left  = self.pruneTree(root.left)
            root.right = self.pruneTree(root.right)
            if not root.left and not root.right and root.val == 0:
                return None
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {TreeNode}
     */
    var pruneTree = function(root) {
        if (!root) return null;
        root.left  = pruneTree(root.left);
        root.right = pruneTree(root.right);
        if (!root.left && !root.right && root.val === 0) return null;
        return root;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion.

## 易错点

- **顺序必须是后序**: 写成前序"先判后递归"对这题不行 —— 进入时根本不知道"整子树是不是全 0", 没法做决定. 必须递归回来后, 用 left/right 的剪枝结果做判断. 这是和 [0669](../0669-trim-a-binary-search-tree/README.md) 最大的对比.
- **判删的三个 AND 都要写齐**: `!root->left && !root->right && root->val == 0` —— 三者缺一不可. 漏掉 val 检查会把"叶子 1 节点"也删掉, 漏掉孩子 null 检查会把"自己是 0 但孩子里有 1"也删掉.
- **整棵树都剪光**: 顶层 root 自己被剪后 return null, 最终结果是 null. 题目允许 (空树合法). 调用方一定要接住 return.
- **不是 trim 是 prune**: trim (0669) 是按区间剪左右; prune 这里是按"全 0"剪整棵子树. 两题名字接近, 别混.

## 相关题目

- [0669. Trim a Binary Search Tree](../0669-trim-a-binary-search-tree/README.md) — 同款树删除通用模板, 但前序版 (判删用自己)
- [0450. Delete Node in a BST](../0450-delete-node-in-a-bst/README.md) — 同款 child-assignment, 删单点要找替身
- [0701. Insert into a BST](../0701-insert-into-a-binary-search-tree/README.md) — 同款 child-assignment, 反向插入
- [0112. Path Sum](../0112-path-sum/README.md) — 同款"用孩子返回判自己" 的后序合并
- [0098. Validate BST](../0098-validate-binary-search-tree/README.md) — 同款 inorder+pre 后序合并风格
- 0865. Smallest Subtree with all Deepest Nodes (待补) — 后序合并的进阶应用

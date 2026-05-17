# 0701. Insert into a Binary Search Tree / 二叉搜索树中的插入操作

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/insert-into-a-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Insert `val` into a BST and return the (possibly new) root. Any valid resulting BST shape is accepted — no rebalancing required.

**中文**: 把 `val` 插入 BST, 返回 (可能新的) 根. 任何合法的 BST 形状都算对, 不要求平衡.

## 思路

### 一句话总结

**操作类 BST: 单向走到底, 撞到空槽就 new 一个返回. 递归用"`root.child = insert(child, val)`"模式自然把新节点挂上去 —— 每层调用都返回自己 (或新建的节点), 上层赋值给对应孩子.**

[0530 口诀](../0530-minimum-absolute-difference-in-bst/README.md#key-insights): 找/插/删 BST 用二分方向, 不全遍历.

## Solution

### Variant A — recursion

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* insertIntoBST(TreeNode* root, int val) {
            if (!root) return new TreeNode(val);   // 空槽 → 新建节点向上返回
            if (val < root->val) {
                root->left  = insertIntoBST(root->left,  val);
            } else {
                root->right = insertIntoBST(root->right, val);
            }
            return root;   // 自己没动, 把自己返回给上层 (上层的 `root->X = ...` 赋回去无副作用)
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def insertIntoBST(self, root: 'TreeNode | None', val: int) -> 'TreeNode':
            if not root:
                return TreeNode(val)
            if val < root.val:
                root.left  = self.insertIntoBST(root.left,  val)
            else:
                root.right = self.insertIntoBST(root.right, val)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} val
     * @return {TreeNode}
     */
    var insertIntoBST = function(root, val) {
        if (!root) return new TreeNode(val);
        if (val < root.val) root.left  = insertIntoBST(root.left,  val);
        else                root.right = insertIntoBST(root.right, val);
        return root;
    };
    ```

### Variant B — iterative (no stack needed)

BST 走一条路径, 维护 `cur` 和 `parent`, 走到空就在 `parent` 上挂新节点.

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* insertIntoBST(TreeNode* root, int val) {
            if (!root) return new TreeNode(val);
            TreeNode* cur = root;
            TreeNode* parent = nullptr;
            while (cur) {
                parent = cur;
                cur = (val < cur->val) ? cur->left : cur->right;
            }
            // 在 parent 的对应孩子位置挂新节点
            if (val < parent->val) parent->left  = new TreeNode(val);
            else                   parent->right = new TreeNode(val);
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def insertIntoBST(self, root: 'TreeNode | None', val: int) -> 'TreeNode':
            if not root:
                return TreeNode(val)
            cur, parent = root, None
            while cur:
                parent = cur
                cur = cur.left if val < cur.val else cur.right
            if val < parent.val:
                parent.left  = TreeNode(val)
            else:
                parent.right = TreeNode(val)
            return root
    ```

=== "JavaScript"
    ```javascript
    var insertIntoBST = function(root, val) {
        if (!root) return new TreeNode(val);
        let cur = root, parent = null;
        while (cur) {
            parent = cur;
            cur = val < cur.val ? cur.left : cur.right;
        }
        if (val < parent.val) parent.left  = new TreeNode(val);
        else                  parent.right = new TreeNode(val);
        return root;
    };
    ```

## Complexity

- **Time**: O(h) — 单向走一条路径; 平衡 O(log n), 退化 O(n).
- **Space**: 递归 O(h); 迭代 O(1).

## 易错点

- **递归靠"`child = insert(...)`"传递新节点**: 关键模式. 当 child 是空槽时, `insert(null, val)` 返回 new node, 上层赋值给 `root->left/right` 就把它挂上去; 当 child 非空时, 递归内部更新孩子, 返回的还是同一个 child 指针, 赋值无副作用. 这种"无论改没改, 都把子树根赋回去" 是树修改类递归的通用骨架, 0450 (delete) 也是同款.
- **递归版别返回 `nullptr`**: `return root` 在所有非空入口都得有, 否则上一层的 `root->left/right` 会被赋成 null, 整棵子树被砍掉. Yang 的代码最后那行 `return root` 必不可少.
- **`<` vs `<=`**: 题目里"任意合法 BST 都算对", 所以等值放左还是右都行. Yang 用 `val < root->val → 左`, 其他都进右 (含等值). 严格 BST 不允许相等, 但 LeetCode 这题宽松, 不用纠结.
- **空树入口**: `root == null` 时返回新节点; 别忘了, 否则递归第一层会解空指针.
- **迭代版要维护 parent**: 因为最后挂节点要赋给 `parent->left/right`, 走到空时 `cur` 没用了, 必须有 `parent` 记着上一步. 不维护 parent 就没法回挂.

## 相关题目

- [0700. Search in a Binary Search Tree](../0700-search-in-a-binary-search-tree/README.md) — 同款 BST 单向走
- [0235. Lowest Common Ancestor of a BST](../0235-lowest-common-ancestor-of-a-binary-search-tree/README.md) — 同款单向走 + 三种比较情况
- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — BST 性质入门
- [0450. Delete Node in a BST / 删除二叉搜索树中的节点](../0450-delete-node-in-a-bst/README.md) — 同款 child-assignment 递归模式, 但删除时要重组子树
- [0108. Convert Sorted Array to Binary Search Tree / 将有序数组转换为二叉搜索树](../0108-convert-sorted-array-to-binary-search-tree/README.md) — 反向: 从数组构造平衡 BST

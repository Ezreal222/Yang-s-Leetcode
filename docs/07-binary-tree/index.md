# 07 · Binary Tree / 二叉树

**EN**: Traversals (DFS/BFS), recursion templates, common tree problems. Grouped by Carl's 6 sub-categories from 代码随想录.

**中文**: 遍历 (DFS/BFS)、递归模板、二叉树常见题。按代码随想录的 6 大分类组织。

## 1 · 遍历方式 / Traversals

DFS three orders + BFS level-order. The whole BFS family (right view, row max/avg, next pointers) extends 0102's level-order template.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0144 | [Binary Tree Preorder Traversal / 二叉树的前序遍历](./0144-binary-tree-preorder-traversal/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0145 | [Binary Tree Postorder Traversal / 二叉树的后序遍历](./0145-binary-tree-postorder-traversal/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0094 | [Binary Tree Inorder Traversal / 二叉树的中序遍历](./0094-binary-tree-inorder-traversal/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0102 | [Binary Tree Level Order Traversal / 二叉树的层序遍历](./0102-binary-tree-level-order-traversal/README.md) | Medium | ✅ | ☑ ☑ ☑ |
| 0107 | [Binary Tree Level Order Traversal II / 二叉树的层序遍历 II](./0107-binary-tree-level-order-traversal-ii/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0199 | [Binary Tree Right Side View / 二叉树的右视图](./0199-binary-tree-right-side-view/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0429 | [N-ary Tree Level Order Traversal / N 叉树的层序遍历](./0429-n-ary-tree-level-order-traversal/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0515 | [Find Largest Value in Each Tree Row / 在每个树行中找最大值](./0515-find-largest-value-in-each-tree-row/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0637 | [Average of Levels in Binary Tree / 二叉树的层平均值](./0637-average-of-levels-in-binary-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0116 | [Populating Next Right Pointers in Each Node / 填充每个节点的下一个右侧节点指针](./0116-populating-next-right-pointers-in-each-node/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0117 | [Populating Next Right Pointers in Each Node II / 填充每个节点的下一个右侧节点指针 II](./0117-populating-next-right-pointers-in-each-node-ii/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0173 | [Binary Search Tree Iterator / 二叉搜索树迭代器](./0173-binary-search-tree-iterator/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0103 | [Binary Tree Zigzag Level Order Traversal / 二叉树的锯齿形层序遍历](./0103-binary-tree-zigzag-level-order-traversal/README.md) | Medium | ✅ | ☑ ☐ ☐ |

## 2 · 属性 / Tree Properties

Compute/check structural properties of a binary tree — depth, balance, symmetry, paths, leaves.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0101 | [Symmetric Tree / 对称二叉树](./0101-symmetric-tree/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0104 | [Maximum Depth of Binary Tree / 二叉树的最大深度](./0104-maximum-depth-of-binary-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0559 | [Maximum Depth of N-ary Tree / N 叉树的最大深度](./0559-maximum-depth-of-n-ary-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0111 | [Minimum Depth of Binary Tree / 二叉树的最小深度](./0111-minimum-depth-of-binary-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0222 | [Count Complete Tree Nodes / 完全二叉树的节点个数](./0222-count-complete-tree-nodes/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0110 | [Balanced Binary Tree / 平衡二叉树](./0110-balanced-binary-tree/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0257 | [Binary Tree Paths / 二叉树的所有路径](./0257-binary-tree-paths/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0404 | [Sum of Left Leaves / 左叶子之和](./0404-sum-of-left-leaves/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0513 | [Find Bottom Left Tree Value / 找树左下角的值](./0513-find-bottom-left-tree-value/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0112 | [Path Sum / 路径总和](./0112-path-sum/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0129 | [Sum Root to Leaf Numbers / 求根节点到叶节点数字之和](./0129-sum-root-to-leaf-numbers/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0113 | [Path Sum II / 路径总和 II](./0113-path-sum-ii/README.md) | Medium | ✅ | ☑ ☐ ☐ |

## 3 · 修改与构造 / Modification & Construction

Build new trees, merge trees, invert / prune / delete-subtree operations. The `root.child = recurse(root.child, ...)` 通用模板.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0226 | [Invert Binary Tree / 翻转二叉树](./0226-invert-binary-tree/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0106 | [Construct Binary Tree from Inorder and Postorder Traversal / 从中序与后序遍历序列构造二叉树](./0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0105 | [Construct Binary Tree from Preorder and Inorder Traversal / 从前序与中序遍历序列构造二叉树](./0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0654 | [Maximum Binary Tree / 最大二叉树](./0654-maximum-binary-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0617 | [Merge Two Binary Trees / 合并二叉树](./0617-merge-two-binary-trees/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0814 | [Binary Tree Pruning / 二叉树剪枝](./0814-binary-tree-pruning/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 1325 | [Delete Leaves With a Given Value / 删除给定值的叶子节点](./1325-delete-leaves-with-a-given-value/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 1110 | [Delete Nodes And Return Forest / 删点成林](./1110-delete-nodes-and-return-forest/README.md) | Medium | ✅ | ☑ ☐ ☐ |

## 4 · 求 BST 的属性 / BST Properties

Use BST's inorder = sorted property to extract aggregates (min diff, mode, greater-sum, validity). "遍历 BST 用中序" 口诀的主战场.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0700 | [Search in a Binary Search Tree / 二叉搜索树中的搜索](./0700-search-in-a-binary-search-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0098 | [Validate Binary Search Tree / 验证二叉搜索树](./0098-validate-binary-search-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0530 | [Minimum Absolute Difference in BST / 二叉搜索树的最小绝对差](./0530-minimum-absolute-difference-in-bst/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0501 | [Find Mode in Binary Search Tree / 二叉搜索树中的众数](./0501-find-mode-in-binary-search-tree/README.md) | Easy | ✅ | ☑ ☑ ☐ |
| 0538 | [Convert BST to Greater Tree / 把二叉搜索树转换为累加树](./0538-convert-bst-to-greater-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0230 | [Kth Smallest Element in a BST / 二叉搜索树中第 K 小的元素](./0230-kth-smallest-element-in-a-bst/README.md) | Medium | ✅ | ☑ ☐ ☐ |

## 5 · 公共祖先问题 / LCA

LCA in general tree (post-order merge) vs LCA in BST (single-walk via property).

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0236 | [Lowest Common Ancestor of a Binary Tree / 二叉树的最近公共祖先](./0236-lowest-common-ancestor-of-a-binary-tree/README.md) | Medium | ✅ | ☑ ☑ ☐ |
| 0235 | [Lowest Common Ancestor of a Binary Search Tree / 二叉搜索树的最近公共祖先](./0235-lowest-common-ancestor-of-a-binary-search-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |

## 6 · BST 的修改与构造 / BST Modification & Construction

Search/insert/delete trio + sorted-input → BST construction. "操作 BST 用二分" 口诀的主战场.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0701 | [Insert into a Binary Search Tree / 二叉搜索树中的插入操作](./0701-insert-into-a-binary-search-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0450 | [Delete Node in a BST / 删除二叉搜索树中的节点](./0450-delete-node-in-a-bst/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0669 | [Trim a Binary Search Tree / 修剪二叉搜索树](./0669-trim-a-binary-search-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |
| 0108 | [Convert Sorted Array to Binary Search Tree / 将有序数组转换为二叉搜索树](./0108-convert-sorted-array-to-binary-search-tree/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0109 | [Convert Sorted List to Binary Search Tree / 有序链表转换二叉搜索树](./0109-convert-sorted-list-to-binary-search-tree/README.md) | Medium | ✅ | ☑ ☐ ☐ |

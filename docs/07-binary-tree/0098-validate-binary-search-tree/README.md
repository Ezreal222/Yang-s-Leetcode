# 0098. Validate Binary Search Tree / 验证二叉搜索树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/validate-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Decide whether a binary tree is a valid BST — for every node, **all** values in the left subtree are strictly less, **all** values in the right subtree are strictly greater.

**中文**: 判断一棵二叉树是不是合法 BST —— 每个节点, **整棵左子树**值都严格小, **整棵右子树**值都严格大.

## 思路

### Core idea

**关键洞察: BST 的中序遍历是严格递增序列**. 所以验证 BST = 中序走一遍, 维护一个 `pre` 指针, 检查每个新访问的节点是不是严格大于 `pre`. 一旦发现不增, 整棵树作废.

### Key Insights

1. **"局部检查"不够 / Local check is not enough**

    新手陷阱: 只比较 `root.left.val < root.val < root.right.val`. 这只查了**直接孩子**, 但 BST 要求**整棵子树**满足. 比如:

    ```
        5
       / \
      1   6
         / \
        3   7    ← 3 < 5 但出现在右子树里, 不合法
    ```

    局部检查会放过它. 必须走中序 (整棵树的全局序) 或者带边界递归.

2. **BST 中序 ⇒ 严格递增, 是 BST 题的通用钥匙 / Inorder of BST is strictly increasing**

    这条等价于 BST 定义本身. 后面 0230 (kth smallest), 0501 (mode), 0530 (min absolute diff) 都是同款 "走 inorder + 处理相邻对" 套路.

3. **`>=` 不是 `>`** —— 题目要求**严格**小于/大于, 相等也算非法.

### 一句话总结

**BST 中序 = 严格递增数列. 写个中序 DFS, 维护 `pre`, 任意时刻 `pre.val >= cur.val` 就返 false. 别用"只比 root 跟左右孩子"那种局部检查 —— 会漏跨层违反.**

## Solution

### Variant A — inorder + `pre` 指针 (Yang's)

中序遍历 + 全局/类成员 `pre` 记录上一个访问的节点.

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* pre = nullptr;
        bool isValidBST(TreeNode* root) {
            if (!root) return true;
            // 左子树合法吗 (顺便更新 pre 到左子树最右那个节点)
            if (!isValidBST(root->left)) return false;
            // 中序访问当前: 跟 pre 比
            if (pre && pre->val >= root->val) return false;
            pre = root;
            // 递归右子树
            return isValidBST(root->right);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isValidBST(self, root: 'TreeNode | None') -> bool:
            self.pre = None
            def inorder(node) -> bool:
                if not node:
                    return True
                if not inorder(node.left):
                    return False
                if self.pre and self.pre.val >= node.val:
                    return False
                self.pre = node
                return inorder(node.right)
            return inorder(root)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {boolean}
     */
    var isValidBST = function(root) {
        let pre = null;
        const inorder = (node) => {
            if (!node) return true;
            if (!inorder(node.left)) return false;
            if (pre && pre.val >= node.val) return false;
            pre = node;
            return inorder(node.right);
        };
        return inorder(root);
    };
    ```

### Variant B — bounds propagation / 带边界递归

每次递归带 `(lo, hi)` 区间: 当前节点必须严格在 `(lo, hi)` 之间; 递归左子树时把上界收紧为 `root.val`, 递归右子树时把下界收紧为 `root.val`.

=== "C++"
    ```cpp
    class Solution {
    public:
        // 用 long long 防止节点值取到 INT_MAX/MIN 时边界对比溢出
        bool helper(TreeNode* node, long long lo, long long hi) {
            if (!node) return true;
            if (node->val <= lo || node->val >= hi) return false;
            return helper(node->left, lo, node->val)
                && helper(node->right, node->val, hi);
        }
        bool isValidBST(TreeNode* root) {
            return helper(root, LLONG_MIN, LLONG_MAX);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isValidBST(self, root: 'TreeNode | None') -> bool:
            # Python 整数天然无上限, 直接用 float('-inf') / float('inf') 当哨兵.
            def helper(node, lo, hi) -> bool:
                if not node:
                    return True
                if node.val <= lo or node.val >= hi:
                    return False
                return (helper(node.left, lo, node.val)
                        and helper(node.right, node.val, hi))
            return helper(root, float('-inf'), float('inf'))
    ```

=== "JavaScript"
    ```javascript
    var isValidBST = function(root) {
        // -Infinity / Infinity 是 JS 内置的浮点哨兵, 任何数都比它大/小
        const helper = (node, lo, hi) => {
            if (!node) return true;
            if (node.val <= lo || node.val >= hi) return false;
            return helper(node.left, lo, node.val) && helper(node.right, node.val, hi);
        };
        return helper(root, -Infinity, Infinity);
    };
    ```

## Complexity

- **Time**: O(n) — both variants visit each node once.
- **Space**: O(h) recursion.

## 易错点

- **"只比 root 跟左右孩子"是错的**: 见 Key Insight #1. 必须用 inorder 或 bounds 来覆盖**整棵子树**的约束.
- **`>=` 不是 `>`**: BST 不允许相等. 写成 `pre.val > root.val` 或 `node.val < lo / > hi` (非严格) 会通过有重复值的反例.
- **C++ bounds 要用 `long long`**: 节点值可能是 `INT_MIN` 或 `INT_MAX`, 直接用 `INT_MIN/MAX` 当上下界, 比较时会"刚好等于"而误判. 升到 `long long` 防溢.
- **Yang's 原版的微优化**: 你写的 `bool left = isValidBST(root->left); ... return left && right;` 是正确的, 但即使 left 已经是 false, 右子树照样会递归一遍. 改成"左 false 立刻 return false"短路掉, 失败用例上快不少 —— 上面 Variant A 已是这个改进版.
- **类成员 `pre` 的副作用**: 把 `pre` 作为类成员 (Yang 的写法) 在 LeetCode 单次提交里没问题, 但**复用同一个 Solution 对象**多次跑就会带上次的状态. 工程代码或在线评测的 multi-test 环境要在入口重置 `pre = nullptr`, 或者把 `pre` 改成函数参数 / 闭包变量.

## 相关题目

- [0700. Search in a Binary Search Tree](../0700-search-in-a-binary-search-tree/README.md) — BST 性质入门
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 提供 inorder 模板; 这题就在它的访问位置加判断
- 0230. Kth Smallest Element in a BST (待补) — inorder 数到第 k 个
- [0501. Find Mode in BST / 二叉搜索树中的众数](../0501-find-mode-in-binary-search-tree/README.md) — inorder 上做众数计数
- [0530. Minimum Absolute Difference in BST / 二叉搜索树的最小绝对差](../0530-minimum-absolute-difference-in-bst/README.md) — inorder 相邻对差的最小值
- [0701. Insert into a BST / 二叉搜索树中的插入操作](../0701-insert-into-a-binary-search-tree/README.md) — 用 BST 性质找插入位置
- [0450. Delete Node in a BST / 删除二叉搜索树中的节点](../0450-delete-node-in-a-bst/README.md) — 用 BST 性质找删除点

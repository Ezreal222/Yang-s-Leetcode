# 0669. Trim a Binary Search Tree / 修剪二叉搜索树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/trim-a-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a BST and a range `[low, high]`, remove every node whose value is **outside** the range. Return the new root of the trimmed BST (relative order preserved).

**中文**: 给一棵 BST 和闭区间 `[low, high]`, 把所有**不在区间内**的节点删掉, 返回修剪后 BST 的新根 (相对结构保持不变).

## 思路

### Core idea

**用 BST 性质 + child-assignment 递归一次走完**:

- `root.val < low` → 整个左子树**全都比 low 还小**, 必删. 但右子树里**可能还有合法值**, 所以**整体替换**为 `trimBST(root.right, ...)` 的结果.
- `root.val > high` → 对称: 右子树整体丢, 返回 `trimBST(root.left, ...)`.
- 在区间内 → root 留, 递归修剪两个孩子.

### Key Insights

1. **Yang's 树删除通用模板 / Universal tree-deletion template**

    所有"删一个/删一片节点"的树问题都套这个壳:

    ```cpp
    TreeNode* dfs(TreeNode* root, ...) {
        if (!root) return nullptr;
        root->left  = dfs(root->left,  ...);   // 处理左, 可能返回 null
        root->right = dfs(root->right, ...);   // 处理右
        if (要删自己) return 替代节点;          // 把上层挂的位置替换掉
        return root;                            // 否则原样挂回
    }
    ```

    递归返回值 = "处理完后这棵子树的新根". 上层用 `root->left = recurse(root->left, ...)` 把它挂回去 —— 自动支持"原样保留"、"换一个子节点顶替"、"砍掉变 null" 三种情况, 不用手动断指针. 这是 [0450 Delete](../0450-delete-node-in-a-bst/README.md) / [0701 Insert](../0701-insert-into-a-binary-search-tree/README.md) / 本题 / 0814 Pruning 等一整族问题的共同骨架.

2. **删不删取决于"自己" → 前序; 取决于"子树状态" → 后序 / Pre vs post by where the predicate lives**

    - 本题判删根据 `root.val` 跟 `[low, high]` 比较, 是**自己的属性** → 在递归**之前**判, 等价**前序**风格. 这样能跳过整个超界子树, 一刀切剪.
    - 像"删除值为 0 的子树 (0814)", 判删要看整个子树是不是全 0, 信息在**孩子里** → 必须先递归再判, 等价**后序**风格.

3. **BST 性质让前序判删变剪枝 / BST property turns the test into pruning**

    `root.val < low` 时, 因为 BST 性质, 左子树**全部** < root.val < low, 一整片直接丢, 不用进去看. 这一刀比"先递归再删每一个超界节点"省掉一整棵 O(子树大小) 的扫描. 普通二叉树没这个 luxury.

### 一句话总结

**用 BST 性质前序判删: `<low` 时整棵自己 + 左子树丢, 直接 return 修剪后的右子树; `>high` 对称; 在区间内就递归两边再挂回. 这是树删除通用模板的"前序剪枝特化版".**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* trimBST(TreeNode* root, int low, int high) {
            if (!root) return nullptr;
            // root 太小: 整个左子树 + 自己一并丢, 答案在修剪后的右子树
            if (root->val < low)  return trimBST(root->right, low, high);
            // root 太大: 整个右子树 + 自己一并丢
            if (root->val > high) return trimBST(root->left,  low, high);
            // 在区间内: 递归修剪两个孩子, 自己留下
            root->left  = trimBST(root->left,  low, high);
            root->right = trimBST(root->right, low, high);
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def trimBST(self, root: 'TreeNode | None', low: int, high: int) -> 'TreeNode | None':
            if not root:
                return None
            if root.val < low:
                return self.trimBST(root.right, low, high)
            if root.val > high:
                return self.trimBST(root.left,  low, high)
            root.left  = self.trimBST(root.left,  low, high)
            root.right = self.trimBST(root.right, low, high)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} low
     * @param {number} high
     * @return {TreeNode}
     */
    var trimBST = function(root, low, high) {
        if (!root) return null;
        if (root.val < low)  return trimBST(root.right, low, high);
        if (root.val > high) return trimBST(root.left,  low, high);
        root.left  = trimBST(root.left,  low, high);
        root.right = trimBST(root.right, low, high);
        return root;
    };
    ```

## Complexity

- **Time**: O(n) worst case — 每个节点最多访问一次. BST 性质让"超界子树整片跳过", 实际节点数远少于 n.
- **Space**: O(h) recursion.

## 易错点

- **`<low` 时返回 `trimBST(root->right, ...)` 不是 `root->right`**: 右子树本身也可能有超界节点, 必须递归. 直接 return `root->right` 会留下污染.
- **不要双重否定**: 看到 "删 < low 的" 容易想成 "保留 ≥ low 的", 然后写成 `if (root->val >= low) return trimBST(...)`. 容易绕. 直接按"什么时候删, 怎么替换"写更清楚.
- **跟 [0450 Delete](../0450-delete-node-in-a-bst/README.md) 对比**: 0450 是删**一个**节点, 要找替身 (in-order successor) 维持 BST 结构; 这里是删**一片**节点 (整个超界子树), 直接用现成的合法子树替换. 两题都用 child-assignment 递归, 但 0450 case 3 复杂得多.
- **题目隐含 `low ≤ high`**: 不需要单独检查. 如果 `low > high` 整棵树都该删, 函数会自然 return null (所有节点都触发 `<low` 或 `>high`, 每条路径递归到底返回 null).

## 相关题目

- [0450. Delete Node in a BST](../0450-delete-node-in-a-bst/README.md) — 同款 child-assignment 模板, 但要找替身
- [0701. Insert into a BST](../0701-insert-into-a-binary-search-tree/README.md) — 同款 child-assignment 模板, 反向: 增节点
- [0700. Search in a BST](../0700-search-in-a-binary-search-tree/README.md) — BST 单向走入门
- [0098. Validate BST](../0098-validate-binary-search-tree/README.md) — BST 性质入门
- [0814. Binary Tree Pruning / 二叉树剪枝](../0814-binary-tree-pruning/README.md) — 普通树版本删全 0 子树, 用后序判删
- 0108. Convert Sorted Array to BST (待补) — 反向: 由数组造平衡 BST

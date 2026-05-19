# 0538. Convert BST to Greater Tree / 把二叉搜索树转换为累加树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/convert-bst-to-greater-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: In a BST, replace each node's value with the sum of all values **≥ that value** (including itself).

**中文**: 把 BST 的每个节点值, 替换成原树里所有**大于等于它**的节点值之和 (含自己).

## 思路

### Core idea

**反中序 = BST 降序遍历** (右 → 中 → 左). 沿降序顺序维护累加 `sum`, 走到每个节点先把 `sum += cur.val`, 然后把 `cur.val = sum`. 因为是降序走的, 此刻 `sum` 正好是 "所有 ≥ cur.val 的节点值之和".

### Key Insights

1. **反中序 (右 → 中 → 左) = BST 严格降序 / Reverse inorder = BST descending**

    BST inorder 是升序; 反过来走 (先 right, 再 mid, 再 left) 就是降序. 任何"按值从大到小遍历 BST"的需求都可以套这个反中序模板, **不用真的把值排序**.

2. **属于 BST inorder + 跨节点聚合家族 / Inorder-aggregate family on BST**

    跟下面这些题同一个骨架 ([0530 Key Insights](../0530-minimum-absolute-difference-in-bst/README.md#key-insights) 提过的"遍历 BST 用中序"):

    | 题目 | 走的方向 | 累加什么 |
    |---|---|---|
    | [0094 Inorder](../0094-binary-tree-inorder-traversal/README.md) | 中序 (升序) | 节点值 → 数组 |
    | [0098 Validate BST](../0098-validate-binary-search-tree/README.md) | 中序 | `pre.val < cur.val` 检查 |
    | [0530 Min Abs Diff](../0530-minimum-absolute-difference-in-bst/README.md) | 中序 | `min(diff, cur - pre)` |
    | [0501 Find Mode](../0501-find-mode-in-binary-search-tree/README.md) | 中序 | 同值连续计数 |
    | **0538 (本题)** | **反中序** (降序) | **累加和, 反向覆盖到节点** |

    都是"按 BST 自然序扫一遍, 顺手做点跨节点聚合". 走向 (中序/反中序) + 累加器 (sum/min/pre/count) 一换就是新题.

3. **`sum` 必须是跨递归共享 / Sum needs to outlive each call**

    递归之间需要传递累加结果, 所以 `sum` 是类成员 (C++) 或闭包/实例属性 (Python/JS). 不能写成局部变量 —— 每次递归就重置成 0 了.

### 一句话总结

**反中序 (右 → 中 → 左) 走 BST = 降序看每个值. 一路累加 `sum`, 把每个节点的值改成"到这里为止的累加和". 这就是"≥ 当前值的所有节点之和".**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int sum = 0;   // 跨递归共享
        void dfs(TreeNode* cur) {
            if (!cur) return;
            dfs(cur->right);          // 先走更大的那边
            sum += cur->val;          // 累加当前
            cur->val = sum;           // 覆写: 现在 sum 是"所有 ≥ 原 cur->val" 的总和
            dfs(cur->left);           // 再走更小的那边
        }
        TreeNode* convertBST(TreeNode* root) {
            dfs(root);
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def convertBST(self, root: 'TreeNode | None') -> 'TreeNode | None':
            self.sum = 0
            def dfs(node):
                if not node:
                    return
                dfs(node.right)           # 大的先访问
                self.sum += node.val
                node.val = self.sum
                dfs(node.left)
            dfs(root)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {TreeNode}
     */
    var convertBST = function(root) {
        let sum = 0;                      // 闭包变量, 跨 dfs 调用共享
        const dfs = (node) => {
            if (!node) return;
            dfs(node.right);
            sum += node.val;
            node.val = sum;
            dfs(node.left);
        };
        dfs(root);
        return root;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion.

## 易错点

- **顺序必须是 右 → 中 → 左**: 任何一个换位都会破坏"sum 等于所有 ≥ cur.val 的总和" 这个不变量. 写成正中序会让 sum 累计的是 ≤ cur.val, 得到的是"小于等于和", 完全是另一道题 (虽然也是合法题: 把每个节点换成 ≤ 它的总和).
- **`sum` 别声明成局部 int**: 每次递归压栈, 局部变量被重置. 必须用类成员 (C++) / 闭包 / 实例属性 (Python/JS) 让它在所有递归调用间共享.
- **原地修改 vs 返回新树**: 题目允许原地改 `cur->val`, 不需要建新节点. 上面代码就是原地版.
- **题目变体: 把每个节点换成"严格大于"的和**: 把累加顺序调成"先访问 right, **然后**累加, 但**先**覆盖再加" 之类 —— 仔细看题, 严格 > vs ≥ 决定累加和写在覆盖前还是覆盖后.

## 相关题目

- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 中序模板, 本题的"反过来"
- [0098. Validate BST](../0098-validate-binary-search-tree/README.md) — 中序 + 跨节点比较
- [0530. Minimum Absolute Difference in BST](../0530-minimum-absolute-difference-in-bst/README.md) — 中序 + 相邻差
- [0501. Find Mode in BST](../0501-find-mode-in-binary-search-tree/README.md) — 中序 + 同值计数
- 1038. Binary Search Tree to Greater Sum Tree (待补) — 完全同款代码, 题面换了句话

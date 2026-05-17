# 0112. Path Sum / 路径总和

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/path-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a tree and a `targetSum`, return whether **any** root-to-leaf path's values sum to `targetSum`.

**中文**: 给一棵树和 `targetSum`, 判断是否**存在**一条从根到叶子的路径, 节点值之和等于 `targetSum`.

## 思路

### Core idea

DFS 一路下走, 每到一个节点就把 `targetSum -= node.val`; 到叶子时检查是否恰好减到 0. 任一子树返回 true 就立刻向上传播 (短路).

### Key Insights

1. **递归减法 vs 累加器 / Subtract-down beats accumulate-up**

    不需要额外的 `curSum` 参数 —— 直接把 `targetSum` 一路减下去, 到叶子比 `targetSum == node.val` 即可. 少一个状态变量, 函数签名也干净.

2. **递归返回值的三种情形 (Carl 框架) / Return value pattern**

    | 场景 | 例子 | 要不要返回值 |
    |---|---|---|
    | 搜索整棵树, 不处理返回值 | [0113 Path Sum II](#) (待补) — 收集所有合格路径 | ❌ void |
    | 搜索整棵树, 要处理/合并返回值 | 0236 LCA — 合并左右子树结果 | ✅ 必须 |
    | 找**一条**符合条件的路径 | **本题** — 找到就立刻返回 true | ✅ 必须 (用来短路) |

    本题属于第三种: 用返回值实现"一旦找到就一路传 true 上去, 兄弟分支不再展开".

### 一句话总结

**DFS, `targetSum` 一路减下去, 叶子处比一下相等就 true. 用返回值短路 —— 任一子树报喜立刻传上去.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool hasPathSum(TreeNode* root, int targetSum) {
            if (!root) return false;
            // 叶子节点: 看看剩下的目标和是不是正好等于本节点值
            if (!root->left && !root->right) return targetSum == root->val;
            // 任一子树找到就 true (短路 ||)
            return hasPathSum(root->left,  targetSum - root->val)
                || hasPathSum(root->right, targetSum - root->val);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def hasPathSum(self, root: 'TreeNode | None', targetSum: int) -> bool:
            if not root:
                return False
            if not root.left and not root.right:
                return targetSum == root.val
            # Python 的 or 也是短路求值, 跟 C++ || 一样
            return (self.hasPathSum(root.left,  targetSum - root.val) or
                    self.hasPathSum(root.right, targetSum - root.val))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} targetSum
     * @return {boolean}
     */
    var hasPathSum = function(root, targetSum) {
        if (!root) return false;
        if (!root.left && !root.right) return targetSum === root.val;
        return hasPathSum(root.left,  targetSum - root.val)
            || hasPathSum(root.right, targetSum - root.val);
    };
    ```

## Complexity

- **Time**: O(n) — 最坏情况下访问每个节点.
- **Space**: O(h) recursion stack.

## 易错点

- **空树返回 false**: 题目里 root 可能是 null, 不能算"路径和 = 0 的合法路径". 入口先 `if (!root) return false`.
- **叶子判定**: `!left && !right` 才是叶子. 对**只有一个孩子的中间节点**, 不能在那里就比 sum, 否则会得到错误答案 (路径还没走完到叶子).
- **不要把 `if (!root)` 当成"路径和 == 0 时返回 true"** —— 这是新手常见的 off-by-one. 真正的判定要在叶子节点做.
- **0113 完全不是同一种递归形态**: 那题要收集**所有**满足的路径, 用 void 递归 + 全局 res + 显式回溯 (path push/pop). 本题只用看"有没有"一条, 短路就行 —— 别套错模板.

## 相关题目

- [0257. Binary Tree Paths](../0257-binary-tree-paths/README.md) — 同款"根到叶子 DFS", 收集字符串
- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 同款 root-to-leaf 递归
- 0113. Path Sum II (待补) — 收集**所有**符合的路径, void 递归 + 显式回溯
- 0129. Sum Root to Leaf Numbers (待补) — 路径累乘积当数字
- 0437. Path Sum III (待补) — 任意起点终点, 双重 DFS 或前缀和
- [0236. Lowest Common Ancestor of a Binary Tree / 二叉树的最近公共祖先](../0236-lowest-common-ancestor-of-a-binary-tree/README.md) — Key Insight 表里的"返回值要处理"那种递归

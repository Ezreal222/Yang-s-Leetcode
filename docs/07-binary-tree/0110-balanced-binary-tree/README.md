# 0110. Balanced Binary Tree / 平衡二叉树

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/balanced-binary-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Decide whether every subtree's left/right heights differ by **at most 1**.

**中文**: 判断二叉树是否平衡 (任意节点的左右子树高度差 ≤ 1).

## 思路

### 一句话总结

**后序 DFS 返回高度, 用 `-1` 作"已经不平衡"哨兵 —— 任一子树不平衡就一路 -1 上去, 短路掉剩余计算. 把朴素的 O(n²) (每节点都跑一遍 height) 拍到 O(n).**

朴素做法: 在每个节点调用一遍 `height(left)` 和 `height(right)`, 比较差值, 然后递归 —— 每个节点的 height 被它所有祖先各算一次, 总共 O(n²).

聪明做法: **一边算 height 一边检查平衡性**. 高度信息一次成形, 检测到不平衡立刻返回 -1, 父节点看到 -1 就直接也返回 -1, 避免任何重复计算 → O(n).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // 返回子树高度; 若已经发现不平衡则返回 -1 短路向上
        int getHeight(TreeNode* cur) {
            if (!cur) return 0;
            int leftHeight = getHeight(cur->left);
            if (leftHeight == -1) return -1;             // 左不平衡, 直接传染
            int rightHeight = getHeight(cur->right);
            if (rightHeight == -1) return -1;            // 右不平衡, 直接传染
            // 当前节点检查 |L - R| > 1 → 也不平衡
            return abs(leftHeight - rightHeight) > 1
                ? -1
                : 1 + max(leftHeight, rightHeight);
        }
        bool isBalanced(TreeNode* root) {
            return getHeight(root) != -1;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isBalanced(self, root: 'TreeNode | None') -> bool:
            def height(node) -> int:
                if not node:
                    return 0
                lh = height(node.left)
                if lh == -1:
                    return -1
                rh = height(node.right)
                if rh == -1:
                    return -1
                if abs(lh - rh) > 1:
                    return -1
                return 1 + max(lh, rh)
            return height(root) != -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {boolean}
     */
    var isBalanced = function(root) {
        const height = (node) => {
            if (!node) return 0;
            const lh = height(node.left);
            if (lh === -1) return -1;
            const rh = height(node.right);
            if (rh === -1) return -1;
            if (Math.abs(lh - rh) > 1) return -1;
            return 1 + Math.max(lh, rh);
        };
        return height(root) !== -1;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点只被算一次高度.
- **Space**: O(h) recursion stack.

## 易错点

- **别写朴素的 O(n²)**: 那种"在每个节点调 `height(left), height(right)` 然后递归"的写法, 大测试数据会 TLE. 用 `-1` 哨兵 + 后序边算边判才是正解.
- **顺序: 算左 → 检查左 -1 → 算右 → 检查右 -1 → 检查 |L-R|**: 早 return -1 是优化, 但顺序写错会让某些不平衡子树被忽略.
- **C++ 三元表达式可读性**: `return abs(L - R) > 1 ? -1 : 1 + max(L, R);` 一行解决, 但写 4 个 `if` 一样对.
- **JS 用 `===` 比较 `-1`**: 严格相等避免类型转换坑.

## 相关题目

- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 这里的 `height` 函数和 0104 一模一样, 多了平衡判断
- [0111. Minimum Depth of Binary Tree](../0111-minimum-depth-of-binary-tree/README.md) — 同款"递归算高度"骨架
- [0226. Invert Binary Tree](../0226-invert-binary-tree/README.md) — 同款递归骨架, 不同操作
- 0543. Diameter of Binary Tree (待补) — 类似套路: 一边算 height 一边维护"经过这点的最长路径"

# 0101. Symmetric Tree / 对称二叉树

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, BFS, Recursion · 二叉树, 深度优先搜索, 广度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/symmetric-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Decide whether a binary tree is a mirror of itself (left subtree mirrors right subtree).

**中文**: 判断一棵二叉树是不是镜像对称的 (左子树是不是右子树的镜像).

## 思路

### Core idea

**比较的不是左右孩子, 而是两棵子树**: 写一个 `compare(L, R)`, 检查 `L` 这棵子树和 `R` 这棵子树是不是互为镜像. 入口调用 `compare(root.left, root.right)`.

镜像的条件展开成 5 种 case:

| L | R | val 比较 | 结论 |
|---|---|---|---|
| null | null | — | ✅ true |
| null | not null | — | ❌ false |
| not null | null | — | ❌ false |
| not null | not null | L.val ≠ R.val | ❌ false |
| not null | not null | L.val = R.val | 递归: **outer** `compare(L.left, R.right)` AND **inner** `compare(L.right, R.left)` |

最后一行是关键: **外侧** = L 的左孩子 vs R 的右孩子, **内侧** = L 的右孩子 vs R 的左孩子. 这就是"镜像"的本质.

### 一句话总结

**写个 `compare(L, R)` 比"两棵子树是不是互为镜像", 入口 `compare(root.left, root.right)`. 镜像的递归定义: 外侧 (L.left, R.right) AND 内侧 (L.right, R.left).**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool compare(TreeNode* left, TreeNode* right) {
            // 5 种 case 顺次排除, 顺序很重要 —— 后面的分支假设前面已排除
            if (left && !right)             return false;
            else if (!left && right)        return false;
            else if (!left && !right)       return true;
            else if (left->val != right->val) return false;
            // 都非空且值相等: 外侧 + 内侧都对称才算
            return compare(left->left,  right->right)
                && compare(left->right, right->left);
        }
        bool isSymmetric(TreeNode* root) {
            if (!root) return true;
            return compare(root->left, root->right);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isSymmetric(self, root: 'TreeNode | None') -> bool:
            def compare(l, r) -> bool:
                if not l and not r:
                    return True
                if not l or not r:           # 一边空一边不空 —— 用 or 一行覆盖两种情况
                    return False
                if l.val != r.val:
                    return False
                return compare(l.left, r.right) and compare(l.right, r.left)
            return not root or compare(root.left, root.right)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {boolean}
     */
    var isSymmetric = function(root) {
        const compare = (l, r) => {
            if (!l && !r) return true;
            if (!l || !r) return false;     // 一边空一边不空
            if (l.val !== r.val) return false;
            return compare(l.left, r.right) && compare(l.right, r.left);
        };
        return !root || compare(root.left, root.right);
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点最多被访问一次.
- **Space**: O(h) recursion stack (h = height; 平衡 O(log n), 退化 O(n)).

## 易错点

- **递归参数传错**: outer 是 `(L.left, R.right)`, inner 是 `(L.right, R.left)`. 写成 `(L.left, R.left)` 就成了"判两棵树是否相同 (0100)" 而不是"是否对称".
- **null 的 5 种 case 必须穷举**: 漏掉"一边 null 一边非 null"会漏返回 false → 一路递归对 nullptr 的 `->val` 解引用直接 crash. Python/JS 同理 IndexError / 报错.
- **顺序敏感**: 上面 5 个 if 的顺序不能乱. `(L 非空, R 空)` 必须先于 `(L 空, R 空)` 这种 catch-all 判断.
- **迭代版本**: 用队列/栈, 每次成对 push `(L, R)` 进来比较. 思路和递归一样, 多了个手动管理的容器.
- **关联视角**: 一棵树对称 ⟺ 它等于它自己的镜像翻转. 所以也可以先调 [0226 invert](../0226-invert-binary-tree/README.md) 再和原树比是否相同 (调用 0100). 不过那是 O(2n) 时间 + 一份额外内存, 不如直接 compare.

## 相关题目

- [0226. Invert Binary Tree / 翻转二叉树](../0226-invert-binary-tree/README.md) — 对称的"反操作": 镜像翻转
- 0100. Same Tree (待补) — 几乎一样, 只是递归方向不同 (`(L.left, R.left)` 而不是 `(L.left, R.right)`)
- 0572. Subtree of Another Tree (待补) — Same Tree 的扩展: 在大树里找子树
- [0104. Maximum Depth](../0104-maximum-depth-of-binary-tree/README.md) — 同款"双递归 + 合并结果"骨架

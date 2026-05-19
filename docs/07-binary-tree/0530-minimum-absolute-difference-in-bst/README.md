# 0530. Minimum Absolute Difference in BST / 二叉搜索树的最小绝对差

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-absolute-difference-in-bst/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: In a BST, find the **minimum absolute difference** between any two node values.

**中文**: BST 中, 求**任意两个节点值**的最小绝对差.

## 思路

### Core idea

**BST inorder = 升序**, 所以全局最小差一定出现在**相邻两个**中序节点之间 (相邻才可能最近, 隔得越远差越大). 走一遍中序, 维护 `pre`, 实时比 `cur.val - pre.val` 取最小.

### Key Insights

1. **BST 的两种用法 —— 遍历 vs 操作 / Two halves of the BST toolkit**

    实战口诀: **"遍历 BST 用中序, 操作 BST 用二分"**.

    | 类别 | 模式 | 例子 |
    |---|---|---|
    | **遍历类** — 要看所有/大部分节点 | inorder DFS, 利用"输出有序"性质 | [0094](../0094-binary-tree-inorder-traversal/README.md), [0098](../0098-validate-binary-search-tree/README.md), [0530 (本题)](../0530-minimum-absolute-difference-in-bst/README.md), 0230, 0501 |
    | **操作类** — 找一个 / 插一个 / 删一个 | 比较 `val` 决定走左还是走右 (BST 二分) | [0700](../0700-search-in-a-binary-search-tree/README.md), 0701, 0450 |

    看到 BST 题, 先问: 我要看所有节点吗? 答 yes → 中序; 答 no → 二分.

2. **最小差一定出现在中序相邻对 / Min diff is always between adjacent pairs**

    数学事实: 升序数列里, 全局最小相邻差就是全局最小差 (任意非相邻对的差 ≥ 它们之间的某个相邻差). 所以**只**比相邻对就够了, 不用维护"所有见过的值".

### 一句话总结

**BST 中序 = 升序; 最小绝对差只可能出现在中序相邻两个之间. 中序 DFS + `pre` 指针, 一路 `min(res, cur - pre)`.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int res = INT_MAX;
        TreeNode* pre = nullptr;
        void traversal(TreeNode* cur) {
            if (!cur) return;
            traversal(cur->left);
            if (pre) res = min(res, cur->val - pre->val);   // 相邻中序节点差
            pre = cur;
            traversal(cur->right);
        }
        int getMinimumDifference(TreeNode* root) {
            traversal(root);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def getMinimumDifference(self, root: 'TreeNode') -> int:
            # 用一个 list 当可变容器闭包变量 (因为 Python 闭包写普通 int 要 nonlocal).
            # 这里 res 和 pre 都用 list-of-one 包起来, 修改 [0] 就行.
            res = [float('inf')]
            pre = [None]
            def inorder(node):
                if not node:
                    return
                inorder(node.left)
                if pre[0] is not None:
                    res[0] = min(res[0], node.val - pre[0].val)
                pre[0] = node
                inorder(node.right)
            inorder(root)
            return res[0]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var getMinimumDifference = function(root) {
        let res = Infinity;
        let pre = null;
        const inorder = (node) => {
            if (!node) return;
            inorder(node.left);
            if (pre) res = Math.min(res, node.val - pre.val);
            pre = node;
            inorder(node.right);
        };
        inorder(root);
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 中序访问每个节点一次.
- **Space**: O(h) recursion.

## 易错点

- **`cur.val - pre.val` 不需要 abs**: 因为 BST 中序严格递增 (题目保证 BST), `cur.val > pre.val` 一定成立, 减出来本来就是正数. 写 `abs(cur.val - pre.val)` 也对但是多余.
- **`pre = nullptr` 哨兵初始**: 处理第一个被访问的节点 —— 它没有"前一个"可以比, 跳过即可. 之后 `pre` 永远非空.
- **Python 闭包改外层 int**: Python 的闭包默认是只读, 要写 outer scope 的变量得用 `nonlocal` 或者把变量包在可变容器 (list / dict / 对象属性) 里, 然后通过下标/属性改. 上面用 `list-of-one` 的小 trick 是常见做法.
- **类成员状态**: 跟 [0098](../0098-validate-binary-search-tree/README.md) 一样, C++ 用 `int res = INT_MAX; TreeNode* pre = nullptr;` 当成员, 多次复用同一个 Solution 实例会带上次的状态. LeetCode 单次提交无所谓, 工程代码要在入口重置.

## 相关题目

- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — 同款 inorder + pre 骨架, 改"min" 为"是否严格递增"
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 提供 inorder 模板
- [0700. Search in a Binary Search Tree](../0700-search-in-a-binary-search-tree/README.md) — BST 性质入门
- [0501. Find Mode in BST / 二叉搜索树中的众数](../0501-find-mode-in-binary-search-tree/README.md) — 同款 inorder + 相邻对处理 (这里换成众数计数)
- 0230. Kth Smallest Element in a BST (待补) — 同款 inorder, 数到第 k 个就 return
- 0783. Minimum Distance Between BST Nodes (待补) — 跟本题完全一样, 只是题面换了句话

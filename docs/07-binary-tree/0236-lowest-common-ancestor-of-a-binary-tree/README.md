# 0236. Lowest Common Ancestor of a Binary Tree / 二叉树的最近公共祖先

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, LCA · 二叉树, 深度优先搜索, 递归, 最近公共祖先
    - **Link**: [LeetCode](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Find the **lowest (deepest) common ancestor** of two given nodes `p` and `q` in a **general** binary tree (no BST property, no parent pointers). Both `p` and `q` are guaranteed to exist.

**中文**: 在**普通**二叉树里找两个节点 `p` 和 `q` 的**最近公共祖先** (没有 BST 性质, 也没有父指针). 保证 `p`、`q` 都存在.

## 思路

### Core idea

**后序递归 + 返回值处理**: 自底向上一路返回"我或我子树里看到了哪些目标". 当一个节点同时收到"左子树报告找到一个" + "右子树报告找到另一个", **这个节点就是 LCA** —— 因为它是第一个让 p 和 q 分别落在两个子树里的祖先.

如果 p、q 都在同一边, 那一边返回的就是答案 (更深处那个早被认作 LCA), 直接传上去.

### Key Insights

1. **走整棵树 + 处理返回值 / Visit-all + merge-return**

    属于 [0112 Key Insights](../0112-path-sum/README.md#key-insights) 表里**第二种**递归模式: 搜索整棵树, **必须处理返回值**, 用返回值在父节点处汇总左右子树的"发现". 跟 0112 (找一条路径就短路) 形成对比 —— 这里**不能短路**, 必须左右都跑.

2. **Base case 同时承担两个角色 / Base case is dual-purpose**

    `if (!root || root == p || root == q) return root;`

    - `!root` → 这条路走到尽头, 没看见 p/q.
    - `root == p` or `root == q` → 看见了**一个**目标, 把这个目标自己作为"信号"传上去.

    **重点**: 即使一个节点是 `p` 的祖先 (不是 p 本身), 它也只把"我子树里有 p"通过返回值反馈; 真正的 LCA 由父节点根据左右子树的返回综合判定.

3. **三种返回组合 / Three return patterns**

    | 左返回 | 右返回 | 当前节点该返回什么 | 含义 |
    |---|---|---|---|
    | non-null | non-null | **`root` (LCA found!)** | p 和 q 分居左右子树, root 就是 LCA |
    | non-null | null | `left` | 答案在左 (可能是 p, 或更深处的 LCA) |
    | null | non-null | `right` | 答案在右 |
    | null | null | `null` | 这棵子树跟 p/q 无关 |

    一旦 LCA 被确定 (case 1), 这个值一路沿调用栈往上传, 不会被覆盖 —— 因为更上层只看左右**子树**的返回, LCA 已经在某个子树里了, 它在上层只会作为唯一的非空返回向上传.

### 一句话总结

**后序递归. Base case 撞到 `null / p / q` 就向上传; 当前节点根据左右两侧返回值汇总: 两侧都非空 → 我是 LCA; 一侧非空 → 把那侧传上去. 关键是不能短路, 必须左右都跑完.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
            // Base: null 或撞到目标本身
            if (!root || root == p || root == q) return root;

            // 后序: 先两边都跑完
            TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
            TreeNode* right = lowestCommonAncestor(root->right, p, q);

            // 汇总
            if (left && right) return root;   // p、q 分居两侧 → 当前就是 LCA
            return left ? left : right;        // 否则把非空那侧传上去
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
            if not root or root is p or root is q:
                return root
            left  = self.lowestCommonAncestor(root.left,  p, q)
            right = self.lowestCommonAncestor(root.right, p, q)
            if left and right:
                return root
            # `left or right`: Python 短路 or 直接返回第一个 truthy 值,
            # 两个都 None 时返回 None. 等价 C++ `left ? left : right`.
            return left or right
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {TreeNode} p
     * @param {TreeNode} q
     * @return {TreeNode}
     */
    var lowestCommonAncestor = function(root, p, q) {
        if (!root || root === p || root === q) return root;
        const left  = lowestCommonAncestor(root.left,  p, q);
        const right = lowestCommonAncestor(root.right, p, q);
        if (left && right) return root;
        // `left || right`: JS 短路 ||, 跟 Python 的 `or` 同款.
        // 第一个 truthy 就返回它; 全部 falsy 返回最后一个 (这里就是 null).
        return left || right;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion.

## 易错点

- **不能短路 (跟 0112 区别)**: 这里**必须**先递归左**再**递归右, 然后才能判断当前是不是 LCA. 写成 `if (!left) return right;` 之类提前 return 会跳过右子树的搜索, 漏掉"另一目标在右"的情况. 0112 找一条路径找到就 return 是另一种递归模式 (见 0112 的 Key Insights 表).
- **Base case 要把 p/q "自己"也算上**: 即使 `root == p`, 你仍然返回 `root` (= p). 上层就根据"左/右子树是否非空"来判断当前 p 是不是同时也是 q 的祖先. 这种"把自己当作发现的信号"的写法是后序递归的常用招式.
- **保证 p、q 都存在**: 题目保证两者都在树中, 所以不需要额外检查"找不到"的情况. 否则要小心: 单边非空时返回那一边, 可能只找到了一个目标但另一个根本不在树里 (返回值会让调用者以为找到了 LCA).
- **`==` vs equality of values**: 比较的是**节点指针/引用**, 不是值. 因为题目里值唯一, `node.val == p.val` 也对; 但工程代码统一用 `node is p` (Python) / `node === p` (JS) / `node == p` (C++ pointer) 更稳, 避免值重复时翻车.
- **BST 版本 0235 简单多了**: BST 有大小性质, 不需要走两边 —— 只需要根据 `p.val`、`q.val` 跟 `root.val` 的关系决定走哪边, O(h) 但只走一条路径. 跟这里的"必须两边都跑"形成鲜明对比.

## 相关题目

- 0235. Lowest Common Ancestor of a Binary Search Tree (待补) — BST 版, 用大小性质单向走, 不需要后序合并
- [0112. Path Sum](../0112-path-sum/README.md) — Carl 框架第三种 (找一条路径短路); 0236 是第二种 (走整棵 + 合并返回)
- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — 也是"后序 + 处理返回值"模式
- [0110. Balanced Binary Tree](../0110-balanced-binary-tree/README.md) — 同款后序 + 返回值短路 (用 -1 哨兵)
- 0865. Smallest Subtree with all the Deepest Nodes (待补) — LCA 的变体: 最深叶子的 LCA
- 1644. LCA II (待补) — p、q 不一定存在的版本, 要额外计数

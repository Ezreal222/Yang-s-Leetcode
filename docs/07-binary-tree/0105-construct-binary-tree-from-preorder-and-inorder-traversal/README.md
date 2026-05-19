# 0105. Construct Binary Tree from Preorder and Inorder Traversal / 从前序与中序遍历序列构造二叉树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, Divide and Conquer, Hash Map · 二叉树, 深度优先搜索, 递归, 分治, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given preorder and inorder traversals (no duplicates), reconstruct the binary tree.

**中文**: 给前序和中序遍历数组 (无重复值), 重建二叉树.

## 思路

### 一句话总结

**[0106](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) 的对偶版 —— 把"postorder 末尾是根"换成"**preorder 开头是根**". 区间约定、切两边数组的同步逻辑、hashmap 优化全部复用.**

跟 0106 的唯一差别:

| 步骤 | 0106 (postorder + inorder) | 0105 (preorder + inorder) |
|---|---|---|
| 根的位置 | `postorder[postorderEnd - 1]` (末尾) | `preorder[preorderBegin]` (开头) |
| 左 X 区间 | `[postorderBegin, +leftSize)` | `[preorderBegin + 1, +1+leftSize)` (跳过根) |
| 右 X 区间 | `[+leftSize, postorderEnd - 1)` (去掉末尾根) | `[+1+leftSize, preorderEnd)` |

`leftSize = delimiterIndex - inorderBegin` —— 跟 0106 一字不差.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // 全程 [begin, end) 半开区间, 同 0106
        TreeNode* traversal(vector<int>& preorder, int preorderBegin, int preorderEnd,
                            vector<int>& inorder,  int inorderBegin,  int inorderEnd) {
            if (preorderBegin == preorderEnd) return nullptr;

            // preorder 开头 = 根 (注意是 preorderBegin 不是 0!)
            int rootValue = preorder[preorderBegin];
            TreeNode* root = new TreeNode(rootValue);

            if (preorderEnd - preorderBegin == 1) return root;

            // 在 inorder 里找根 (注意索引 inorder, 不是 preorder!)
            int delimiterIndex;
            for (delimiterIndex = inorderBegin; delimiterIndex < inorderEnd; delimiterIndex++) {
                if (inorder[delimiterIndex] == rootValue) break;
            }

            // 切 inorder
            int leftInorderBegin  = inorderBegin;
            int leftInorderEnd    = delimiterIndex;
            int rightInorderBegin = delimiterIndex + 1;
            int rightInorderEnd   = inorderEnd;

            // 切 preorder: 跳过开头的根, 左半长度 = 左 inorder 长度
            int leftSize = delimiterIndex - inorderBegin;
            int leftPreorderBegin  = preorderBegin + 1;
            int leftPreorderEnd    = preorderBegin + 1 + leftSize;
            int rightPreorderBegin = leftPreorderEnd;
            int rightPreorderEnd   = preorderEnd;

            root->left  = traversal(preorder, leftPreorderBegin,  leftPreorderEnd,
                                    inorder,  leftInorderBegin,   leftInorderEnd);
            root->right = traversal(preorder, rightPreorderBegin, rightPreorderEnd,
                                    inorder,  rightInorderBegin,  rightInorderEnd);
            return root;
        }
        TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
            return traversal(preorder, 0, preorder.size(), inorder, 0, inorder.size());
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def buildTree(self, preorder: list[int], inorder: list[int]) -> 'TreeNode | None':
            # value → index 哈希表, 找根 O(1) → 总复杂度 O(n)
            idx = {v: i for i, v in enumerate(inorder)}

            def build(pre_lo: int, pre_hi: int, in_lo: int, in_hi: int) -> 'TreeNode | None':
                if pre_lo == pre_hi:
                    return None
                root_val = preorder[pre_lo]              # 注意 pre_lo, 不是 0
                root = TreeNode(root_val)
                k = idx[root_val]
                left_size = k - in_lo
                root.left  = build(pre_lo + 1, pre_lo + 1 + left_size, in_lo, k)
                root.right = build(pre_lo + 1 + left_size, pre_hi, k + 1, in_hi)
                return root

            return build(0, len(preorder), 0, len(inorder))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} preorder
     * @param {number[]} inorder
     * @return {TreeNode}
     */
    var buildTree = function(preorder, inorder) {
        const idx = new Map();
        inorder.forEach((v, i) => idx.set(v, i));

        const build = (preLo, preHi, inLo, inHi) => {
            if (preLo === preHi) return null;
            const rootVal = preorder[preLo];
            const root = new TreeNode(rootVal);
            const k = idx.get(rootVal);
            const leftSize = k - inLo;
            root.left  = build(preLo + 1,             preLo + 1 + leftSize, inLo,  k);
            root.right = build(preLo + 1 + leftSize,  preHi,                k + 1, inHi);
            return root;
        };
        return build(0, preorder.length, 0, inorder.length);
    };
    ```

## Complexity

- **Time**: O(n) (with hashmap), O(n²) (linear scan in worst skewed tree).
- **Space**: O(n).

## 易错点

- **`preorder[preorderBegin]` 不是 `preorder[0]`**: Yang flagged. 只有第一次入口调用时 `preorderBegin == 0`, 递归进子树就不是了 —— 写 `preorder[0]` 永远拿到整树的根, 子树构造直接错.
- **`inorder[delimiterIndex]` 不是 `preorder[delimiterIndex]`**: 也 flagged. 找根的位置要在 **inorder** 里找; `delimiterIndex` 是 inorder 的下标, 不是 preorder 的.
- **左 preorder 多 +1**: 跟 0106 不同, preorder **开头**是根, 切左 preorder 时要 `+1` 跳过去 (`preorderBegin + 1`). 0106 的 postorder 是**末尾**是根, 所以右 postorder 才需要 `-1`.
- **重复值不能用此法**: 题目保证无重复. 有重复就无法在 inorder 里唯一定位根.
- **0106 + 0105 共有的纪律**: 区间约定全程一致 (这里都是 `[begin, end)`), 区间约定一旦混用立刻全盘错位.

## 相关题目

- [0106. Construct Binary Tree from Inorder and Postorder Traversal](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 对偶版, 完整讨论在那边
- [0654. Maximum Binary Tree / 最大二叉树](../0654-maximum-binary-tree/README.md) — 同款"找根 + 切两半递归"
- [0617. Merge Two Binary Trees / 合并二叉树](../0617-merge-two-binary-trees/README.md) — 同款双递归同步

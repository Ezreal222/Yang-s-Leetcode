# 0106. Construct Binary Tree from Inorder and Postorder Traversal / 从中序与后序遍历序列构造二叉树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, Divide and Conquer, Hash Map · 二叉树, 深度优先搜索, 递归, 分治, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given the inorder and postorder traversals of a binary tree (no duplicate values), reconstruct and return the tree.

**中文**: 给中序和后序遍历数组 (节点值无重复), 重建并返回这棵二叉树.

## 思路

### Core idea

**分治 + 双数组联动切分**:

1. **Postorder 末尾就是当前子树的根** —— 这是后序的本质 ("左 → 右 → 根").
2. 在 inorder 里找到这个根 —— 它把 inorder 切成 **[左子树的 inorder | 根 | 右子树的 inorder]**.
3. 左 inorder 长度 == 左 postorder 长度 (左子树节点数固定) —— 用这个长度把 postorder 也切成 **[左 postorder | 右 postorder | 根]**.
4. 递归两边构建.

### Key Insights

1. **左闭右开区间 `[begin, end)` 全程统一 / Half-open intervals everywhere**

    所有数组切片都用 `[begin, end)` 半开区间, 不混 `[]` 闭区间. 好处:
    - `end - begin` 直接是子数组**长度**.
    - 空区间是 `begin == end`, 一行就能判结束 (`if (postorderEnd == postorderBegin) return nullptr`).
    - 切的时候不用 ±1 来回算.

    **永远统一一种区间约定**, 是这类"在数组上分治"问题最关键的纪律.

2. **左右子树同步切两个数组 / Sync-split both arrays**

    切 inorder 找到 `delimiterIndex` 后, 用**左 inorder 的长度**`= delimiterIndex - inorderBegin` 来切 postorder. 这就是为什么必须用同一种区间约定: 左 postorder 的长度必须跟左 inorder 完全一致.

3. **线性扫找根 → 哈希表优化到 O(n) / Hashmap optimization**

    朴素版每层调用都要 O(子树大小) 找根, 总共 O(n²). **预先**把 inorder 的 `value → index` 存进哈希表, 找根变 O(1), 总复杂度降到 **O(n)**. 当 n 大时一定要这个优化.

### 一句话总结

**Postorder 末尾 = 根, 用根在 inorder 切两半, 同时用左半长度切 postorder, 递归. 全程左闭右开 `[begin, end)`, 找根上 hashmap 把 O(n²) 拍到 O(n).**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // [begin, end) 半开区间, 空区间 = begin == end
        TreeNode* traversal(vector<int>& inorder,   int inorderBegin,   int inorderEnd,
                            vector<int>& postorder, int postorderBegin, int postorderEnd) {
            if (postorderEnd == postorderBegin) return nullptr;

            // postorder 最后一个 = 当前子树的根
            int rootValue = postorder[postorderEnd - 1];
            TreeNode* root = new TreeNode(rootValue);

            // 叶子节点早返回 (微小优化, 可省)
            if (postorderEnd - postorderBegin == 1) return root;

            // 在 inorder 里找根的位置 (O(子树大小); hashmap 优化见 Pitfalls)
            // 注意: delimiterIndex 必须在 for 外面声明, 因为退出循环后还要用它
            int delimiterIndex;
            for (delimiterIndex = inorderBegin; delimiterIndex < inorderEnd; delimiterIndex++) {
                if (inorder[delimiterIndex] == rootValue) break;
            }

            // 切 inorder: 左 [inorderBegin, delimiterIndex), 右 [delimiterIndex+1, inorderEnd)
            int leftInorderBegin  = inorderBegin;
            int leftInorderEnd    = delimiterIndex;
            int rightInorderBegin = delimiterIndex + 1;
            int rightInorderEnd   = inorderEnd;

            // 切 postorder: 左半长度 = delimiterIndex - inorderBegin, 跟左 inorder 同长
            int leftPostorderBegin  = postorderBegin;
            int leftPostorderEnd    = postorderBegin + (delimiterIndex - inorderBegin);
            int rightPostorderBegin = leftPostorderEnd;
            int rightPostorderEnd   = postorderEnd - 1;   // 去掉最后那个根

            root->left  = traversal(inorder, leftInorderBegin,  leftInorderEnd,
                                    postorder, leftPostorderBegin,  leftPostorderEnd);
            root->right = traversal(inorder, rightInorderBegin, rightInorderEnd,
                                    postorder, rightPostorderBegin, rightPostorderEnd);
            return root;
        }
        TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
            return traversal(inorder, 0, inorder.size(), postorder, 0, postorder.size());
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def buildTree(self, inorder: list[int], postorder: list[int]) -> 'TreeNode | None':
            # 预先建立 value → index 的字典, 找根变 O(1) —— 总复杂度 O(n²) → O(n).
            #   C++ 等价: unordered_map<int, int>
            idx = {v: i for i, v in enumerate(inorder)}

            def build(in_lo: int, in_hi: int, po_lo: int, po_hi: int) -> 'TreeNode | None':
                # [in_lo, in_hi) 和 [po_lo, po_hi) 都是半开区间
                if po_lo == po_hi:
                    return None
                root_val = postorder[po_hi - 1]
                root = TreeNode(root_val)
                k = idx[root_val]                    # 根在 inorder 的位置
                left_size = k - in_lo
                root.left  = build(in_lo, k, po_lo, po_lo + left_size)
                root.right = build(k + 1, in_hi, po_lo + left_size, po_hi - 1)
                return root

            return build(0, len(inorder), 0, len(postorder))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} inorder
     * @param {number[]} postorder
     * @return {TreeNode}
     */
    var buildTree = function(inorder, postorder) {
        // Map: 跟 plain object 比, key 可以是任何类型, 查找/插入都是 O(1).
        // 这里 key 是 number, 用 Map 比 {} 更明确语义.
        //   C++ 等价: unordered_map<int, int>
        const idx = new Map();
        inorder.forEach((v, i) => idx.set(v, i));

        const build = (inLo, inHi, poLo, poHi) => {
            if (poLo === poHi) return null;
            const rootVal = postorder[poHi - 1];
            const root = new TreeNode(rootVal);
            const k = idx.get(rootVal);
            const leftSize = k - inLo;
            root.left  = build(inLo, k,    poLo,            poLo + leftSize);
            root.right = build(k + 1, inHi, poLo + leftSize, poHi - 1);
            return root;
        };
        return build(0, inorder.length, 0, postorder.length);
    };
    ```

## Complexity

| Approach | Time | Space |
|---|---|---|
| linear scan to find root | O(n²) worst | O(n) |
| **hashmap lookup** | **O(n)** | O(n) |

均 O(n) 空间 (hashmap + 递归栈).

## 易错点

- **`delimiterIndex` 必须在 for 外声明**: Yang flagged 这个. 写成 `for (int delimiterIndex = ...; ...; ...)` 退出循环后变量出 scope, 后面切数组就用不上了. 在 for 前 `int delimiterIndex;` 声明 + 在 for 里赋初值 + break 出来后用它的值 —— 这是 C 系列语言里专门为"循环找首个匹配下标"设计的写法.
- **区间约定不能混**: 一旦定下 `[begin, end)`, 全程都用半开. 哪怕一个地方写成 `[begin, end]` 闭区间, 切数组的长度算式就错位 1, 整个递归都崩.
- **`postorderEnd - 1` 取根**: 因为 `[begin, end)` 是半开, 最后一个元素的下标是 `end - 1`, 不是 `end`.
- **空区间判定**: `if (postorderEnd == postorderBegin) return nullptr` —— 比单独判 `inorderEnd == inorderBegin` 都行 (它们任何时候同步空), 选一个写就够.
- **节点值有重复**: 这题保证无重复, 所以 inorder 里的根唯一. **如果有重复就不能用这个方法** (无法唯一定位根) —— 真实数据要先验证.
- **0105 (preorder + inorder) 是对偶题**: 思路一模一样, 把"postorder 末尾"换成"preorder 开头". 同套区间约定 + hashmap 优化.

## 相关题目

- [0105. Construct Binary Tree from Preorder and Inorder Traversal / 从前序与中序遍历序列构造二叉树](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 对偶版, 把 postorder 换成 preorder
- [0654. Maximum Binary Tree / 最大二叉树](../0654-maximum-binary-tree/README.md) — 同款"找一个特殊位置切两半递归构造"
- [0617. Merge Two Binary Trees / 合并二叉树](../0617-merge-two-binary-trees/README.md) — 同款双递归同步处理两棵树
- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 这题反向操作: 已知树求遍历
- [0145. Binary Tree Postorder Traversal](../0145-binary-tree-postorder-traversal/README.md) — 同上

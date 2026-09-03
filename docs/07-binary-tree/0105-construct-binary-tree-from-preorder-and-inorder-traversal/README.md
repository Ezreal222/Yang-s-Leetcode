# 0105. Construct Binary Tree from Preorder and Inorder Traversal / 从前序与中序遍历序列构造二叉树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, Divide and Conquer, Hash Map · 二叉树, 深度优先搜索, 递归, 分治, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☑ ☐

## Problem

**EN**: Given preorder and inorder traversals (no duplicates), reconstruct the binary tree.

**中文**: 给前序和中序遍历数组 (无重复值), 重建二叉树.

## 思路

### 一句话总结

**[0106](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) 的对偶版 —— 把"postorder 末尾是根"换成"**preorder 开头是根**". 区间约定、切两边数组的同步逻辑、hashmap 优化全部复用.**

### Core idea

**Preorder 头 = 根**, 用 inorder 里根的位置**切分左右**. 递归构造. 关键优化: **hashmap 存 `inorder value → index`**, 找根 O(1), 总复杂度 **O(n)** (原始线性扫是 O(n²) 最坏).

### Key insights

- **Preorder 首元素定根** — 而 0106 是 postorder 末元素定根.
- **Inorder 里根的左边 = 左子树, 右边 = 右子树** — 这是所有 preorder/postorder + inorder 重建题的共通钥匙.
- **左子树大小 `leftSize = k - inL`** — 一旦知道左子树多长, preorder 就能切: 头是根, `[preL+1, preL+leftSize]` 是左, `[preL+leftSize+1, preR]` 是右.
- **`unordered_map<value, index>` 一次预处理** — 把每步 O(n) 找根降到 O(1).
- **闭区间 `[preL, preR]` vs 半开 `[begin, end)`** — Yang 的新版用**闭区间** + `preL > preR` 判空, 比半开少 1 次 +1 心智负担.

### Transferable thinking

**"Root defines the split"** — 任何一对包含"根位置线索"和"左右分界线索"的两种遍历, 都可以用此模式重建. 也见 [0106](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) (postorder+inorder), [0889 Preorder+Postorder](待补). **切左右子树时, 两个数组必须同步切** —— 左 inorder 长度 = 左 preorder 长度, 这是同步锚点.

### One-liner

> **Preorder 头定根, inorder 位切两半, 左右递归; hashmap 让"找根" O(1).**

### 跟 0106 的对比

| 步骤 | 0106 (postorder + inorder) | 0105 (preorder + inorder) |
|---|---|---|
| 根的位置 | `postorder[postR]` (末尾) | `preorder[preL]` (开头) |
| 左 X 区间 | `[postL, postL + leftSize - 1]` | `[preL + 1, preL + leftSize]` (跳过根) |
| 右 X 区间 | `[postL + leftSize, postR - 1]` | `[preL + leftSize + 1, preR]` |

`leftSize = k - inL` —— 跟 0106 一字不差.

## Solution

=== "C++ (推荐: hashmap + 闭区间 + 指针共享)"
    ```cpp
    class Solution {
    public:
        unordered_map<int, int> pos;        // inorder value → index, O(1) 找根
        vector<int>* pre;                    // 指针避免每次递归传引用, 减栈开销

        // 闭区间 [preL, preR], [inL, inR]
        TreeNode* build(int preL, int preR, int inL, int inR) {
            if (preL > preR) return nullptr;

            int rootVal = (*pre)[preL];      // preorder 头 = 根
            TreeNode* root = new TreeNode(rootVal);

            int k = pos[rootVal];            // 根在 inorder 的位置, O(1)
            int leftSize = k - inL;          // 左子树大小 = 根左边 inorder 长度

            root->left  = build(preL + 1,             preL + leftSize, inL,    k - 1);
            root->right = build(preL + leftSize + 1,  preR,            k + 1,  inR);
            return root;
        }

        TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
            int n = inorder.size();
            for (int i = 0; i < n; i++) pos[inorder[i]] = i;   // 预处理: O(n)
            pre = &preorder;
            return build(0, n - 1, 0, n - 1);
        }
    };
    ```

=== "C++ (v1 教学: 半开区间 + 线性扫)"
    ```cpp
    class Solution {
    public:
        // 全程 [begin, end) 半开区间, 同 0106; 每次线性扫找根 → O(n²) 最坏
        TreeNode* traversal(vector<int>& preorder, int preorderBegin, int preorderEnd,
                            vector<int>& inorder,  int inorderBegin,  int inorderEnd) {
            if (preorderBegin == preorderEnd) return nullptr;

            int rootValue = preorder[preorderBegin];
            TreeNode* root = new TreeNode(rootValue);
            if (preorderEnd - preorderBegin == 1) return root;

            int delimiterIndex;
            for (delimiterIndex = inorderBegin; delimiterIndex < inorderEnd; delimiterIndex++) {
                if (inorder[delimiterIndex] == rootValue) break;
            }

            int leftSize = delimiterIndex - inorderBegin;
            root->left  = traversal(preorder, preorderBegin + 1,
                                    preorderBegin + 1 + leftSize,
                                    inorder,  inorderBegin, delimiterIndex);
            root->right = traversal(preorder, preorderBegin + 1 + leftSize,
                                    preorderEnd,
                                    inorder,  delimiterIndex + 1, inorderEnd);
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
            # dict comprehension: value → index 哈希表, O(1) 找根
            # {v: i for i, v in enumerate(...)} 是 Python 一行建 map 的惯用招
            # 相当于 C++ for (int i = 0; i < n; i++) pos[inorder[i]] = i
            pos = {v: i for i, v in enumerate(inorder)}

            # 闭区间 [preL, preR], [inL, inR]
            def build(preL: int, preR: int, inL: int, inR: int) -> 'TreeNode | None':
                if preL > preR:
                    return None
                root_val = preorder[preL]                # preorder 头 = 根
                root = TreeNode(root_val)
                k = pos[root_val]
                left_size = k - inL
                root.left  = build(preL + 1,             preL + left_size, inL,    k - 1)
                root.right = build(preL + left_size + 1, preR,             k + 1,  inR)
                return root

            return build(0, len(preorder) - 1, 0, len(inorder) - 1)
    ```

=== "JavaScript"
    ```javascript
    var buildTree = function(preorder, inorder) {
        // Map: JS 的哈希表. forEach((v, i) => ...) 同时拿值和索引
        // 相当于 C++ 的 unordered_map<int,int> + for 循环填
        const pos = new Map();
        inorder.forEach((v, i) => pos.set(v, i));

        // 闭区间 [preL, preR], [inL, inR]
        // Arrow function 保留外层 preorder 的闭包访问, 不用传参
        const build = (preL, preR, inL, inR) => {
            if (preL > preR) return null;
            const rootVal = preorder[preL];              // preorder 头 = 根
            const root = new TreeNode(rootVal);
            const k = pos.get(rootVal);
            const leftSize = k - inL;
            root.left  = build(preL + 1,             preL + leftSize, inL,    k - 1);
            root.right = build(preL + leftSize + 1,  preR,            k + 1,  inR);
            return root;
        };
        return build(0, preorder.length - 1, 0, inorder.length - 1);
    };
    ```

## Complexity

- **Time**: O(n) (with hashmap), O(n²) (linear scan in worst skewed tree).
- **Space**: O(n).

## 易错点

- **`preorder[preL]` 不是 `preorder[0]`**: 只有第一次入口 `preL == 0`, 递归进子树就不是了 —— 写 `preorder[0]` 永远拿整树的根, 子树直接错.
- **左子树大小是 `k - inL`, 不是 `k`**: `k` 是根在 inorder 的**绝对下标**, 左子树大小要减去当前段起点 `inL`. 忘减 → 递归越界或死循环.

## 相关题目

- [0106. Construct Binary Tree from Inorder and Postorder Traversal](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 对偶版, 完整讨论在那边
- [0654. Maximum Binary Tree / 最大二叉树](../0654-maximum-binary-tree/README.md) — 同款"找根 + 切两半递归"
- [0617. Merge Two Binary Trees / 合并二叉树](../0617-merge-two-binary-trees/README.md) — 同款双递归同步

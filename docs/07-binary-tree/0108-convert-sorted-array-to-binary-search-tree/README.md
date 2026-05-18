# 0108. Convert Sorted Array to Binary Search Tree / 将有序数组转换为二叉搜索树

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BST, Recursion, Divide and Conquer · 二叉树, 二叉搜索树, 递归, 分治
    - **Link**: [LeetCode](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a sorted (ascending) integer array, build a **height-balanced** BST.

**中文**: 给一个升序数组, 构造一棵**高度平衡**的二叉搜索树.

## 思路

### Core idea

**取中点作根, 左半递归造左子树, 右半递归造右子树**:

- 数组有序 → 中点左边都小于中点, 右边都大于中点 → 自动满足 BST 性质.
- 每次切成"左半 + 中 + 右半", 两边大小最多差 1 → 自动满足"高度平衡" (左右高度差 ≤ 1).

### Key Insights

1. **BST 构造三件套 / Three BST/tree-construction patterns**

    | 题目 | 切分点怎么找 | 输入 |
    |---|---|---|
    | **[0105/0106](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) Build from traversals** | 用另一数组的开头/末尾 (pre/post 的根) | 两个遍历数组 |
    | **[0654](../0654-maximum-binary-tree/README.md) Maximum Binary Tree** | 当前区间的 argmax | 一个数组 |
    | **0108 (本题)** | 当前区间的中点 | 一个**有序**数组 |

    都是"分治 + 递归造左右子树"骨架, 区别只是**切分点怎么选**.

2. **`mid = i + (j - i) / 2` 防溢出 / Overflow-safe midpoint**

    写成 `(i + j) / 2` 在 `i`, `j` 都接近 `INT_MAX` 时会溢出. Yang's 写法是 C++ 二分查找的肌肉记忆 idiom. Python/JS 整数无 32-bit 限制, 但保留这个写法是好习惯.

3. **闭区间 `[i, j]` vs 半开区间 `[begin, end)`**

    Yang 这题用了**闭区间**: `build(nums, 0, nums.size() - 1)`, base case `if (i > j)`. 之前 [0105/0106/0654](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) 用半开 `[begin, end)`. 两种都对, 关键是**一题之内不混用**:

    - 闭区间: 长度 = `j - i + 1`, 空 = `i > j`, 切分 `[i, mid-1]` / `[mid+1, j]`.
    - 半开: 长度 = `end - begin`, 空 = `begin == end`, 切分 `[lo, mid)` / `[mid+1, hi)`.

### 一句话总结

**有序数组取中点当根, 左右半递归. 自动平衡 (每层切一半) + 自动 BST (中点左小右大). `mid = i + (j-i)/2` 防溢出.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // 闭区间 [i, j]
        TreeNode* build(vector<int>& nums, int i, int j) {
            if (i > j) return nullptr;
            int mid = i + (j - i) / 2;           // 防溢出: 不写 (i+j)/2
            TreeNode* root = new TreeNode(nums[mid]);
            root->left  = build(nums, i,       mid - 1);
            root->right = build(nums, mid + 1, j);
            return root;
        }
        TreeNode* sortedArrayToBST(vector<int>& nums) {
            return build(nums, 0, nums.size() - 1);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sortedArrayToBST(self, nums: list[int]) -> 'TreeNode | None':
            def build(i: int, j: int) -> 'TreeNode | None':
                if i > j:
                    return None
                mid = (i + j) // 2               # Python 整数无溢出, 直接写
                root = TreeNode(nums[mid])
                root.left  = build(i,       mid - 1)
                root.right = build(mid + 1, j)
                return root
            return build(0, len(nums) - 1)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {TreeNode}
     */
    var sortedArrayToBST = function(nums) {
        const build = (i, j) => {
            if (i > j) return null;
            // (i + j) >> 1 是位移整除 2, 写起来短; JS 数字精度大, 不会溢出.
            // 但要注意 >> 是 32-bit signed, 数据特别大 (≥ 2³¹) 要用 Math.floor((i+j)/2).
            const mid = (i + j) >> 1;
            const root = new TreeNode(nums[mid]);
            root.left  = build(i,       mid - 1);
            root.right = build(mid + 1, j);
            return root;
        };
        return build(0, nums.length - 1);
    };
    ```

## Complexity

- **Time**: O(n) — 每个元素恰好成为一个节点, 各被访问一次.
- **Space**: O(log n) recursion (平衡保证了高度 ≤ log n + 1) + O(n) 输出树本身.

## 易错点

- **闭区间 base case 是 `i > j` 不是 `i >= j`**: 闭区间 `[i, j]` 空的条件是 i 越过 j (因为 i == j 还有一个元素 nums[i]). 写成 `>=` 会漏掉单元素区间.
- **切左右半要 ±1**: 闭区间下, mid 已经被当前节点用掉, 左半 `[i, mid-1]`, 右半 `[mid+1, j]`. 别忘了 ±1, 否则会无限递归 (传同一个区间下去).
- **空数组**: `nums.size() == 0` 时入口调 `build(nums, 0, -1)`, base case `0 > -1` true → return nullptr. 自然处理, 不需要单独特判. C++ 注意 `nums.size()` 是 `size_t` (unsigned), 转 int 给参数最稳.
- **题目允许多个合法答案**: 取**中点偏左** (`mid = (i+j)/2`) 或**中点偏右** (`mid = (i+j+1)/2`) 都对 —— 选 偶数长度的中间偏哪个不影响"平衡". LeetCode 接受任一合法形状.
- **不要混区间风格**: 别在同一函数里 `[i, mid)` 半开切左 + `(mid, j]` 半开切右. 一种风格走到底.

## 相关题目

- [0105. Construct Binary Tree from Preorder and Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 同款分治构造, 切分点用 preorder 头
- [0106. Construct Binary Tree from Inorder and Postorder](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 同款分治, 切分点用 postorder 尾
- [0654. Maximum Binary Tree](../0654-maximum-binary-tree/README.md) — 同款分治, 切分点用 argmax
- [0700. Search in a BST](../0700-search-in-a-binary-search-tree/README.md) — BST 性质入门
- [0109. Convert Sorted List to Binary Search Tree / 有序链表转换二叉搜索树](../0109-convert-sorted-list-to-binary-search-tree/README.md) — 进阶: 输入是链表, 取中点是 O(n) 不再是 O(1); 三个变体 O(n) → O(n log n) → O(n)
- 1382. Balance a BST (待补) — 先把 BST 转成 sorted array (inorder), 然后用 0108 重建

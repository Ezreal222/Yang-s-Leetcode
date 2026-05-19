# 0654. Maximum Binary Tree / 最大二叉树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, Divide and Conquer, Monotonic Stack · 二叉树, 深度优先搜索, 递归, 分治, 单调栈
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Recursively construct a "maximum binary tree" from `nums`: at each step, the root is the **max value of the current range**; left subtree is built from the elements to the left of max, right subtree from elements to the right.

**中文**: 递归构造"最大二叉树": 当前区间的**最大值**作根, 左边元素递归构造左子树, 右边元素递归构造右子树.

## 思路

### 一句话总结

**[0105/0106](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) 同款的分治构造, 只是"切分点"从"另一个数组里找"换成"**当前区间里找最大值**". 区间约定 `[begin, end)` 全程一致.**

朴素递归 O(n²) (每层扫一遍找 max). 想 O(n) 要单调栈, 见 Pitfalls.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // [begin, end) 半开区间
        TreeNode* traversal(vector<int>& nums, int begin, int end) {
            if (end == begin) return nullptr;
            int maxIdx = begin;                 // 只追踪 idx, 比较时用 nums[maxIdx]
            for (int i = begin; i < end; i++) {
                if (nums[i] > nums[maxIdx]) maxIdx = i;
            }
            TreeNode* root = new TreeNode(nums[maxIdx]);
            root->left  = traversal(nums, begin, maxIdx);     // 左半 [begin, maxIdx)
            root->right = traversal(nums, maxIdx + 1, end);   // 右半 [maxIdx+1, end)
            return root;
        }
        TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
            return traversal(nums, 0, nums.size());
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def constructMaximumBinaryTree(self, nums: list[int]) -> 'TreeNode | None':
            def build(lo: int, hi: int) -> 'TreeNode | None':
                if lo == hi:
                    return None
                # max(range, key=lambda i: nums[i]): 在下标范围里挑使 nums[i] 最大的下标.
                # `key` 让 max 不直接比较元素值, 而是比较 key(元素) 的返回值.
                # 这里就是"在下标空间里, 按 nums[i] 排序找最大".
                #   C++ 等价: for 循环手写 maxIdx = argmax over [lo, hi).
                k = max(range(lo, hi), key=lambda i: nums[i])
                root = TreeNode(nums[k])
                root.left  = build(lo, k)
                root.right = build(k + 1, hi)
                return root
            return build(0, len(nums))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {TreeNode}
     */
    var constructMaximumBinaryTree = function(nums) {
        const build = (lo, hi) => {
            if (lo === hi) return null;
            let k = lo;
            for (let i = lo + 1; i < hi; i++) {
                if (nums[i] > nums[k]) k = i;
            }
            const root = new TreeNode(nums[k]);
            root.left  = build(lo, k);
            root.right = build(k + 1, hi);
            return root;
        };
        return build(0, nums.length);
    };
    ```

## Complexity

- **Naive recursion**: Time O(n²) (skewed worst case), Space O(h).
- **Monotonic stack** (see Pitfalls): Time **O(n)**, Space O(n).

## 易错点

- **`if (end - begin == 1) return root;` 是多余的**: Yang 注意到了. 单元素区间会进入循环找出唯一一个 max, 然后递归两侧都是空区间 (`begin == end`) → 直接 return nullptr. 不写早返回也对, 写了不破坏正确性, 只是冗余.
- **只需要 `maxIdx` 不需要 `maxVal`**: Yang 也注意到了. 比较时通过 `nums[maxIdx]` 间接拿值, 少维护一个变量, 代码更紧凑.
- **区间纪律 `[begin, end)`**: 跟 0105/0106 一致. 切左半是 `[begin, maxIdx)`, 切右半是 `[maxIdx+1, end)` —— `maxIdx` 那个位置不归任何子树管.
- **O(n) monotonic stack 解法**: 维护一个**单调递减栈** (stack 中元素从底到顶递减). 扫到新值 `v`:
    1. 把所有 `< v` 的栈顶弹掉, **最后弹出的那个**变成 `v.left` (它们都在 v 左侧, 而 v 比它们都大).
    2. 弹完后栈非空 → 栈顶的 `right = v` (因为 v 比它小, v 在它右侧).
    3. 把 v 节点压栈.
    每个节点入栈出栈各一次, O(n) 均摊. 这是这题的"高分解法", 思想跟 [0155 Min Stack](../../06-stack-queue/0155-min-stack/README.md) 优化版的单调栈是亲戚.
- **不要用切片在 Python 里**: `nums[lo:hi]` 每次切都是 O(n) 拷贝, 总开销退化成 O(n²) 起步, 大数据 TLE. 用下标参数 `(lo, hi)` 就好.

## 相关题目

- [0105. Construct Binary Tree from Preorder and Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 同款分治, 切分点从另一数组找
- [0106. Construct Binary Tree from Inorder and Postorder](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 同上的对偶
- [0155. Min Stack](../../06-stack-queue/0155-min-stack/README.md) — 单调栈思想入门 (这题的 O(n) 解法用同款思想)
- 0998. Maximum Binary Tree II (待补) — 续作: 在已建好的最大二叉树上插入新值
- 0739. Daily Temperatures (待补) — 单调栈经典训练

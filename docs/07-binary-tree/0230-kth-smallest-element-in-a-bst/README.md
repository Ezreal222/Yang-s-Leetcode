# 0230. Kth Smallest Element in a BST / 二叉搜索树中第 K 小的元素

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Stack, Inorder · 二叉树, 二叉搜索树, 深度优先搜索, 栈, 中序遍历
    - **Link**: [LeetCode](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given a BST and integer k, return the kth smallest value.

**中文**: 给一棵 BST 和整数 k, 返回第 k 小的节点值.

## 思路

### Core idea

**BST 中序 = 升序** → 走 inorder, 数到第 k 个就是答案. 两种实现一回事:

- **递归版**: 全局 `count`, inorder 走到第 k 个时记 `res`, 用 `count > k` guard 提前剪枝.
- **迭代版**: 用 0094/0173 的栈 + 左脊模板, 每弹一个节点 `--k`, k 归零立即返回. **走到答案就停, 不再往下**.

### Key Insights

1. **"遍历 BST 用中序" 的又一例 / Yet another inorder-aggregate on BST**

    跟 [0530](../0530-minimum-absolute-difference-in-bst/README.md) / [0501](../0501-find-mode-in-binary-search-tree/README.md) / [0538](../0538-convert-bst-to-greater-tree/README.md) 一个家族. 不同点: 这题只关心**第 k 个**, 不需要全扫完 → 多了"早停"这一步.

2. **Early termination 是这题的关键 / Visit only k nodes, not n**

    天真写法是"inorder 进 vector 再取 v[k-1]" — O(n) 时间 O(n) 空间. 早停版只访问前 k 个 → 时间 **O(h + k)** (h 是下到最左叶子的开销, k 是从那里弹 k 次), 空间 O(h).

    - **迭代版** `if (--k == 0) return cur->val` 一行直接 return, 最干净.
    - **递归版** 用 `if (count > k) return` 在每次入口拦截 — 找到答案后, 后续递归会进一两层就被拦, 不是真"立即停"但够用.

3. **递归 vs 迭代: 看场景挑 / Pick by call pattern**

    - **一次查询**: 两种都好, 递归版代码短.
    - **频繁查询 + 树常变 (follow-up)**: 给每个节点维护 `size = 1 + size(left) + size(right)`, 单次查询 O(h):
      ```
      if (k == size(left) + 1) return cur->val;
      if (k <  size(left) + 1) recurse left;
      else                     recurse right with k -= size(left)+1;
      ```
      这是把"第 k 小"问题翻成"在哪个子树的哪个位置" — augmented BST 的标准做法.

4. **0173 iterator 视角 / Or just call next() k times**

    如果已经有 [0173](../0173-binary-search-tree-iterator/README.md) 的 BSTIterator, 这题就是:
    ```cpp
    BSTIterator it(root);
    for (int i = 1; i < k; ++i) it.next();
    return it.next();
    ```
    本题的迭代版**就是把 BSTIterator 的逻辑展开内联**.

### 一句话总结

**Inorder 走 BST + 数到 k 个就停. 迭代版栈 + `--k` 最干净, 递归版加个 `count > k` guard 也行.**

## Solution

=== "C++"
    === "迭代 (推荐)"
        ```cpp
        class Solution {
        public:
            int kthSmallest(TreeNode* root, int k) {
                stack<TreeNode*> stk;
                TreeNode* cur = root;
                while (cur || !stk.empty()) {
                    while (cur) {                  // 一路压左脊
                        stk.push(cur);
                        cur = cur->left;
                    }
                    cur = stk.top(); stk.pop();
                    if (--k == 0) return cur->val; // 第 k 个出栈即答案
                    cur = cur->right;
                }
                return -1;                         // unreachable per constraints
            }
        };
        ```

    === "递归"
        ```cpp
        class Solution {
        public:
            int count = 0, res;
            void dfs(TreeNode* cur, int k) {
                if (!cur || count > k) return;     // count > k: 答案已找, 不再下钻
                dfs(cur->left, k);
                if (++count == k) res = cur->val;
                dfs(cur->right, k);
            }
            int kthSmallest(TreeNode* root, int k) {
                dfs(root, k);
                return res;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def kthSmallest(self, root: 'TreeNode', k: int) -> int:
            stk = []                              # list 当 stack
            cur = root
            while cur or stk:
                while cur:                        # 压左脊
                    stk.append(cur)
                    cur = cur.left
                cur = stk.pop()
                k -= 1
                if k == 0:
                    return cur.val
                cur = cur.right
            return -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} k
     * @return {number}
     */
    var kthSmallest = function(root, k) {
        const stk = [];
        let cur = root;
        while (cur || stk.length) {
            while (cur) {
                stk.push(cur);
                cur = cur.left;
            }
            cur = stk.pop();
            if (--k === 0) return cur.val;
            cur = cur.right;
        }
        return -1;
    };
    ```

## Complexity

- **Time**: O(h + k) — 下到最左叶子是 O(h), 然后弹 k 次. 平衡 BST 是 O(log n + k).
- **Space**: O(h) — 栈最多压一条左脊.

## 易错点

- **递归版 guard 不够紧**: `count > k` 拦截在入口, 找到答案后还会多进一两层才退. 想真"立即停"得用异常 / `optional<int>` short-circuit 或者改成 boolean 返回. Yang 这版能过, 但常数偏大.
- **不要先 inorder dump 再取 `v[k-1]`**: 写法对但空间 O(n) + 时间 O(n), 不利用早停. 面试官会问能不能省.
- **`--k == 0` vs `k == 0` 后 `--k`**: 注意先减后比 (`--k`) — 第 k 次弹栈时, k 从 1 减到 0, 命中. 写成 `if (k-- == 0)` 是先比再减, 死循环.
- **题目变体 (follow-up): 频繁修改 + 频繁查询**: 简单 inorder 每次 O(n) 太贵. 上"BST + size 增广", 单次 O(h). 这是经典的 "augmented tree" 套路, [树状数组/线段树第 k 大] 也是同思路.
- **k 是 1-indexed**: 第 1 小就是最小值, 不是第 0 个. 别 off-by-one.

## 相关题目

- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 同款迭代式 inorder 模板
- [0173. Binary Search Tree Iterator](../0173-binary-search-tree-iterator/README.md) — 把这题的迭代版**包装成 iterator**; 反过来这题就是它调 `next()` k 次
- [0530. Minimum Absolute Difference in BST](../0530-minimum-absolute-difference-in-bst/README.md) — 同款"inorder + 跨节点聚合"
- [0501. Find Mode in BST](../0501-find-mode-in-binary-search-tree/README.md) — 同款 inorder + 计数
- [0538. Convert BST to Greater Tree](../0538-convert-bst-to-greater-tree/README.md) — **反**中序的姐妹题: 第 k **大**
- 0700. Search in BST + size 增广 (待补) — augmented BST 单次 O(h) 查 kth
- 0215. Kth Largest in Array (待补) — heap / quickselect, 数组版的 kth

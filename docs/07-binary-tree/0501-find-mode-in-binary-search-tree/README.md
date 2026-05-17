# 0501. Find Mode in Binary Search Tree / 二叉搜索树中的众数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/find-mode-in-binary-search-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Find all **modes** (most frequent values) in a BST. Equal values are allowed; return all values whose frequency ties for max.

**中文**: 找 BST 中所有**众数**(出现频率最高的值). 允许相等值; 返回所有出现频率并列最高的值.

## 思路

### Core idea

**BST inorder = 升序**, 所以**相等的值一定连续相邻**. 走一遍中序, 维护 `(pre, curCount, maxCount, res)`:

- 同一段相等值跑 `curCount++`, 段切换时 `curCount = 1`.
- 每访问完一个节点立刻拿 `curCount` 跟 `maxCount` 比, **一次扫完**得到所有众数, 不用先扫一遍找 max 再扫一遍收集.

### Key Insights

1. **三种 `pre` 状态决定 `curCount` / Three pre states**

    | `pre` | 这一节点要做 | 含义 |
    |---|---|---|
    | `nullptr` (第一次访问) | `curCount = 1` | 新值, 从 1 数起 |
    | `pre.val == cur.val` | `curCount++` | 还在同一段相等值里 |
    | `pre.val != cur.val` | `curCount = 1` | 切到新值, 重新计数 |

    然后**统一**更新 `pre = cur`.

2. **计数逻辑 vs 结果集逻辑是正交的 / Count logic and res logic are orthogonal**

    Yang 的关键提醒. 不要把"更新 res" 塞进 "更新 curCount" 的三个分支里 —— 那样三个分支都要重写一遍 res 处理, 啰嗦还容易写错. 正确做法: **先**只管 curCount (三个分支), **再**用统一的"`curCount` 跟 `maxCount` 比一比" 块更新 res:

    ```text
    if curCount > maxCount:
        maxCount = curCount
        res.clear()       # 之前收集的全废了
        res.push(cur.val)
    elif curCount == maxCount:
        res.push(cur.val) # 新众数, 并列
    # curCount < maxCount: 什么都不做
    ```

    这种"分阶段 + 各管各的"是写复杂状态机/递归时的通用纪律.

3. **BST 性质把 O(n) 空间降到 O(1) extra / BST property saves space**

    如果不利用 BST 的有序性 (比如对一般二叉树), 你得用 `unordered_map<int,int>` 数频次, O(n) 空间. BST 因为相等值连续, 一个 `pre` 指针 + 几个计数器就够了, 额外空间 O(1) (不算递归栈).

### 一句话总结

**BST 中序 → 相等值必相邻. 走一遍中序, 三段式更新 `curCount`, 然后用 `curCount` 跟 `maxCount` 比来动态维护 `res`. 计数逻辑和结果逻辑分开写, 别耦合.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxCount = 0;
        int curCount = 0;
        vector<int> res;
        TreeNode* pre = nullptr;

        void traversal(TreeNode* cur) {
            if (!cur) return;
            traversal(cur->left);

            // 阶段 1: 维护 curCount (三种 pre 情况)
            if (!pre)                       curCount = 1;
            else if (pre->val == cur->val)  curCount++;
            else                            curCount = 1;
            pre = cur;

            // 阶段 2: 用 curCount 跟 maxCount 比, 动态维护 res
            if (curCount > maxCount) {
                maxCount = curCount;
                res.clear();
                res.push_back(cur->val);
            } else if (curCount == maxCount) {
                res.push_back(cur->val);
            }

            traversal(cur->right);
        }
        vector<int> findMode(TreeNode* root) {
            traversal(root);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findMode(self, root: 'TreeNode') -> list[int]:
            self.max_count = 0
            self.cur_count = 0
            self.res: list[int] = []
            self.pre: 'TreeNode | None' = None

            def inorder(node):
                if not node:
                    return
                inorder(node.left)

                # 阶段 1: curCount
                if self.pre is None or self.pre.val != node.val:
                    self.cur_count = 1
                else:
                    self.cur_count += 1
                self.pre = node

                # 阶段 2: res
                if self.cur_count > self.max_count:
                    self.max_count = self.cur_count
                    self.res = [node.val]    # 等价 C++ res.clear() + push_back
                elif self.cur_count == self.max_count:
                    self.res.append(node.val)

                inorder(node.right)

            inorder(root)
            return self.res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number[]}
     */
    var findMode = function(root) {
        let maxCount = 0, curCount = 0;
        let res = [];
        let pre = null;

        const inorder = (node) => {
            if (!node) return;
            inorder(node.left);

            // 阶段 1: curCount
            if (!pre || pre.val !== node.val) curCount = 1;
            else curCount++;
            pre = node;

            // 阶段 2: res
            if (curCount > maxCount) {
                maxCount = curCount;
                res = [node.val];
            } else if (curCount === maxCount) {
                res.push(node.val);
            }

            inorder(node.right);
        };
        inorder(root);
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 中序访问每个节点一次.
- **Space**: O(h) recursion + **O(1) extra** (不算 res 输出). 不利用 BST 性质的 hashmap 版本要 O(n) extra.

## 易错点

- **三种 `pre` 状态别合并**: 看起来 case 1 (`!pre`) 和 case 3 (`pre != cur`) 都是 `curCount = 1`, 可以写成 `if (!pre || pre.val != cur.val)`. 这是合理的合并, Python/JS 版本就这么写的. 但 case 2 (`pre.val == cur.val` → `count++`) 不能跟其他合并.
- **`res.clear()` 必须发生在 `curCount > maxCount` 时**: 一旦发现更高频次, 之前收集的"并列众数"全部作废. 漏 clear 会把过时的并列值留在结果里.
- **C++ `res.clear()` 释放 vector**: 注意 `clear()` 是把 size 归零, capacity 不变 —— 不必担心反复 clear 触发频繁内存分配.
- **Python 重置 res 用 `self.res = [node.val]` 而不是 `.clear() + .append()`**: 两种都对, 前者一行更紧凑. JS 同理.
- **状态机递归** 这类题考验的就是**两块逻辑分阶段写**. Yang 提醒的"不要把 res 处理塞进 pre 三分支" 就是这个意思. 状态多了一定要分阶段, 否则维护爆炸.

## 相关题目

- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — 同款 inorder + pre 骨架, 检查的是"严格递增"
- [0530. Minimum Absolute Difference in BST](../0530-minimum-absolute-difference-in-bst/README.md) — 同款 inorder + pre 骨架, 算的是"相邻差的最小"
- [0094. Binary Tree Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 提供 inorder 模板
- 0230. Kth Smallest Element in a BST (待补) — 同款 inorder, 数到第 k 个 return
- 0653. Two Sum IV - BST (待补) — BST + 双指针 / 哈希

# 0113. Path Sum II / 路径总和 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Backtracking · 二叉树, 深度优先搜索, 回溯
    - **Link**: [LeetCode](https://leetcode.com/problems/path-sum-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Return **all** root-to-leaf paths whose values sum to `targetSum`.

**中文**: 返回**所有**从 root 到 leaf 路径上节点值之和等于 `targetSum` 的路径.

## 思路

### Core idea

DFS, 一路 `targetSum -= cur->val`. 进入节点先把它 push 到 `path`, 到叶子且 `targetSum == cur->val` (扣完这层正好归零) 就把整条 `path` 快照拷贝进 `res`. 退出节点前 `path.pop_back()` — **显式回溯**.

### Key Insights

1. **0112 是 bool, 0113 是收集 → 必须显式回溯 / Detect vs collect**

    [0112](../0112-path-sum/README.md) 只问 "有没有一条", 短路返 `true` 即可. 0113 要找出**所有**, 不能短路, 而且每条都要"在那个时间点的 path 全貌". 答案是引用同一个 `path` 容器 + 进入 push / 退出 pop — 这是回溯系列 (subsets/permutations/combinations) 的基础模板.

2. **回到[0257](../0257-binary-tree-paths/README.md) 的"隐式 vs 显式"对比**

    | 题 | path 类型 | 回溯方式 | 原因 |
    |---|---|---|---|
    | 0257 | `string`, 按值传 | **隐式** (拷贝自动回退) | string 拼接每次都是新对象, 父调用不受影响 |
    | 0129 | `int curSum`, 按值传 | **隐式** | int 同理, 按值传 |
    | **0113 (本题)** | `vector<int>& path`, 按引用共享 | **显式 push/pop** | vector 要 push_back(path) 拷贝快照, 共享比每次拷贝快 |

    选哪种: **快照需求决定**. 0113 push 进 res 的是 path 的当前状态, 之后 path 还要变 → 用共享 + 拷贝快照 + 显式回溯, 不要每次都传值复制.

3. **`path` 必须按引用 (or 全局)**

    如果 `dfs(TreeNode*, int targetSum, vector<int> path)` 按值, 每次递归 O(depth) 拷贝, 总 O(n × depth) = 最坏 O(n²). 共享一个 path + push/pop 才是 O(n) 加上 O(总路径长) 输出.

4. **判定时机: push 之后, 递归之前 / Check after push, before recursing**

    叶子检查放在 push 后, 因为要把当前节点也算进 path. 同 0112 的检查时机一致 ("当前节点是不是叶子 + 当前 targetSum 是否等于自身").

5. **targetSum 减法 vs 累加 / Subtract from target vs accumulate sum**

    - **减法版** (Yang): `dfs(left, targetSum - cur->val)`. 叶子判定 `targetSum == cur->val`. 跟 0112 一致.
    - **累加版**: 维护 `currentSum += cur->val`, 叶子判 `currentSum == targetSum`. 多一个变量但更直观.

    两者等价, 减法版代码短半行. 减法版的 `targetSum` 因为是按值传, 在多个递归分支里互不影响 — 隐式回溯了**这一个变量**, 跟 0129 的 `curSum` 同理.

### 一句话总结

**DFS 减 targetSum, path 显式 push/pop. 叶子且 `targetSum == cur->val` 时把 path 整个快照拷贝进 res.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> res;
        vector<int> path;
        void dfs(TreeNode* cur, int targetSum) {
            if (!cur) return;
            path.push_back(cur->val);
            if (!cur->left && !cur->right && targetSum == cur->val) {
                res.push_back(path);              // 快照拷贝
            }
            dfs(cur->left,  targetSum - cur->val);
            dfs(cur->right, targetSum - cur->val);
            path.pop_back();                      // 显式回溯
        }
        vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
            dfs(root, targetSum);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def pathSum(self, root: 'TreeNode | None', targetSum: int) -> list[list[int]]:
            res, path = [], []
            def dfs(node, remain):
                if not node:
                    return
                path.append(node.val)
                if not node.left and not node.right and remain == node.val:
                    res.append(path[:])           # path[:] 切片 = 拷贝快照, 等价 C++ res.push_back(path)
                dfs(node.left,  remain - node.val)
                dfs(node.right, remain - node.val)
                path.pop()                        # 显式回溯, 对应 C++ pop_back
            dfs(root, targetSum)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} targetSum
     * @return {number[][]}
     */
    var pathSum = function(root, targetSum) {
        const res = [], path = [];
        const dfs = (node, remain) => {
            if (!node) return;
            path.push(node.val);
            if (!node.left && !node.right && remain === node.val) {
                res.push([...path]);              // 扩展运算符 = 浅拷贝快照. 等价 path.slice() 或 Array.from(path)
            }
            dfs(node.left,  remain - node.val);
            dfs(node.right, remain - node.val);
            path.pop();                           // 显式回溯
        };
        dfs(root, targetSum);
        return res;
    };
    ```

## Complexity

- **Time**: O(n) 节点访问 + O(总路径长) 拷贝输出. 退化成链表时输出可达 O(n), 完美二叉树时所有路径长之和 O(n log n).
- **Space**: O(h) recursion + path 最长 h.

## 易错点

- **`path.pop_back()` 必须在 dfs 之后**: 一定要在两个子递归之后再 pop. 写在 dfs 之间 / 之前都会让某个子调用拿到错误的 path.
- **`res.push_back(path)` 必须是拷贝, 不能是引用**: C++ vector push_back 默认就是拷贝, 没问题. JS/Python 默认是 push 引用, 必须显式 `[...path]` / `path[:]`, 否则 res 里全是同一个最后清空的数组.
- **不要短路返回 (return) 在找到一条之后**: 这题要**所有**满足的路径, 找到一条后还要继续找右子树. 同 0112 的写法**不要直接搬**.
- **叶子判定不要漏 `!cur->right`**: 单孩子节点不是叶子. 必须左右都空才算.
- **减法版要在叶子比 `targetSum == cur->val` (不是 `== 0`)**: 因为减法发生在递归调用时, 进入叶子之前 targetSum 还没扣自己. 别写成 `targetSum - cur->val == 0` 在 push 之后 — 那是另一种等价写法但容易绕晕.
- **空根**: `if (!cur) return` 自然处理. 不需特判空树.

## 相关题目

- [0112. Path Sum](../0112-path-sum/README.md) — 同款 root→leaf 累加, 只要 boolean. 这题的"前置版"
- [0257. Binary Tree Paths](../0257-binary-tree-paths/README.md) — 同款收集路径, 但用 string + 隐式回溯
- [0129. Sum Root to Leaf Numbers](../0129-sum-root-to-leaf-numbers/README.md) — 同款 root→leaf, 累加成数字
- 0437. Path Sum III (待补) — 任意起点/终点的路径和, 前缀和 + 哈希
- 0078/0046/0077. Subsets / Permutations / Combinations (待补) — 显式回溯模板的经典训练场, 跟本题完全一个套路

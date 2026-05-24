# 0337. House Robber III / 打家劫舍 III

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Tree DP, Postorder DFS · 动态规划, 树形 DP, 后序 DFS
    - **Link**: [LeetCode](https://leetcode.com/problems/house-robber-iii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 房子排成**二叉树**: 偷直接父子相邻则触发警报. 求最多能偷多少.

**中文**: 二叉树形态, 父子相邻不能同时偷, 求最大偷取金额.

## Key Insights

1. **树形 DP + 后序 DFS / Tree DP via postorder**

    线性 / 环形 ([0198](../0198-house-robber/README.md) / [0213](../0213-house-robber-ii/README.md)) 用一维数组, **树形**就用**后序 DFS** — 自底向上, 子树先算完, 父节点拼装. 这是树形 DP 的标准节奏.

    > 看到"树 + 求最优 + 父子有约束" → 立刻反应"后序 DFS + 每节点返回多状态".

2. **每个节点返回两个状态: {不偷, 偷} / Each node returns (notRob, rob)**

    跟线性 0198 一样, 每个节点有两种"决策", 但树形下需要**显式返回两种值**, 让父节点决定怎么选:

    | 状态 | 值 |
    |---|---|
    | `notRob` | 不偷当前节点能拿到的最大金额 (子节点可偷可不偷) |
    | `rob`    | 偷当前节点能拿到的最大金额 (子节点必须不偷) |

    > **"返回多状态"** 是树形 DP 的标志. 0198 是把两状态藏在 `dp[i]` 滚动里 (`prev1` = 隐含最优), 树形必须显式传上去, 否则父节点无法知道子节点"偷与不偷" 各值多少.

3. **🔑 转移: `rob = val + 左不偷 + 右不偷`; `notRob = max(左) + max(右)` / Transition**

    - **偷当前**: 子节点必须不偷 → `rob = node.val + left.notRob + right.notRob`.
    - **不偷当前**: 子节点自由选 → `notRob = max(left.notRob, left.rob) + max(right.notRob, right.rob)`.

    > 这就是 [§10 DP 思维流程 — 最后一步](../topic-dp-thinking-process.md) 在树上的应用: "**当前节点偷或不偷**, 子树的最优各是多少". 一字不差套.

4. **Base case: 空节点返回 `{0, 0}` / Null returns zero pair**

    空节点偷不偷都得 0. 让递推自然终止.

5. **返回数组 vs `pair` 选哪个 / Vector vs pair**

    Yang 用了 `vector<int>` 返回. C++ 也可以用 `pair<int, int>` (语义稍清晰且栈分配, 不分配堆); 也可以用结构体. 三种都正确, 函数式风格下 `pair` 更主流:

    ```cpp
    pair<int, int> dfs(TreeNode* node) {
        if (!node) return {0, 0};
        auto [ln, lr] = dfs(node->left);
        auto [rn, rr] = dfs(node->right);
        int rob_   = node->val + ln + rn;
        int notRob = max(ln, lr) + max(rn, rr);
        return {notRob, rob_};
    }
    ```

    > 结构化绑定 `auto [a, b] = ...` 是 C++17 解构, 比 `vector` 索引干净.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int rob(TreeNode* root) {
            vector<int> r = dfs(root);
            return max(r[0], r[1]);                                // r = {notRob, rob}
        }
    private:
        vector<int> dfs(TreeNode* node) {
            if (!node) return {0, 0};                              // 空节点: 偷不偷都 0
            vector<int> left  = dfs(node->left);                   // 后序: 先递归子树
            vector<int> right = dfs(node->right);
            int robCur    = node->val + left[0] + right[0];        // 偷当前 → 子节点必须不偷
            int notRobCur = max(left[0], left[1]) + max(right[0], right[1]);
            return {notRobCur, robCur};
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def rob(self, root: 'TreeNode') -> int:
            def dfs(node):
                # 返回 (notRob, rob), 跟 C++ 一致
                if not node:
                    return (0, 0)
                ln, lr = dfs(node.left)                            # 解构赋值, Python 原生
                rn, rr = dfs(node.right)
                rob_cur     = node.val + ln + rn                   # 偷当前 → 子节点不偷
                not_rob_cur = max(ln, lr) + max(rn, rr)            # 不偷 → 子节点随意
                return (not_rob_cur, rob_cur)
            return max(dfs(root))                                  # max(tuple) 取两个里大的
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var rob = function(root) {
        const dfs = (node) => {
            if (!node) return [0, 0];                              // [notRob, rob]
            const [ln, lr] = dfs(node.left);                       // 解构, ES6
            const [rn, rr] = dfs(node.right);
            const robCur    = node.val + ln + rn;
            const notRobCur = Math.max(ln, lr) + Math.max(rn, rr);
            return [notRobCur, robCur];
        };
        return Math.max(...dfs(root));                             // spread + Math.max
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) — 递归栈, 树高 h. 最坏 O(n) (链状树).

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性版
- [0213. House Robber II](../0213-house-robber-ii/README.md) — 环形版
- 0124\. Binary Tree Maximum Path Sum (待补) — 同款"后序 DFS + 多状态返回"
- 0968\. Binary Tree Cameras — 后序 DFS + 三状态, 跟本题同套路 → [§09 0968](../../09-greedy/0968-binary-tree-cameras/README.md)
- [§10 DP 思维流程](../topic-dp-thinking-process.md) — 树形 DP 是"最后一步" 思维在树上的体现

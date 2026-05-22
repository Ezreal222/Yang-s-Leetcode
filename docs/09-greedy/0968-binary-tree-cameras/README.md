# 0968. Binary Tree Cameras / 监控二叉树

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Greedy, DFS, Tree, DP · 贪心, 深度优先, 二叉树, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-cameras/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: A camera at a node monitors its parent, itself, and its immediate children. Return the minimum number of cameras needed to monitor every node.

**中文**: 节点上的摄像头能监控其父节点, 自身, 及左右子节点. 求监控整棵树所需最少摄像头数.

## Key Insights

1. **后序遍历: 自底向上传状态 / Postorder so children report first**

    必须先知道**子节点是否被覆盖**, 才能决定当前节点要不要装摄像头. 所以 DFS 必须**后序** (先递归左右, 再处理当前). 前/中序拿不到子节点的"覆盖状态", 决策没法做.

2. **三状态编码 / Three-state DP**

    每个节点返回三种状态之一:

    | 状态 | 含义 | 父节点该怎么反应 |
    |------|------|------------------|
    | **0** | 未被覆盖 | **必须**装摄像头来覆盖它 |
    | **1** | 被覆盖, 自己无摄像头 | 父节点无须为它装 |
    | **2** | 自己有摄像头 | 父节点被它覆盖, 状态 = 1 |

    决策表 (基于 `left, right` 两个子状态):

    - **任一子 = 0** → 当前必须装 → `cameras++`, 返回 2.
    - **任一子 = 2** → 当前被子覆盖 → 返回 1.
    - **两子都 = 1** → 当前未被覆盖 (子既无摄像头也不能伸手到父) → 返回 0, 让父去装.

    优先级**严格** "先看 0, 再看 2, 否则 0": "两子都被覆盖但都无摄像头" 必须自己上报"未覆盖", 让父辈接手 — 这样能跳跃覆盖更多.

3. **nullptr 必须返回 1 (覆盖) / Null child returns "covered"**

    如果 nullptr 返回 0 (未覆盖), 所有叶子节点的"空孩子" 都会触发"必须装摄像头" → **叶子全装** → 极度浪费.

    把 nullptr 当成"已覆盖, 无摄像头" → 叶子节点收到两个 1 → 返回 0 → **父节点装一个摄像头, 覆盖叶子 + 自己 + 兄弟**, 一举三得.

    > 这是**贪心的核心**: 叶子不装, 装在叶子的父亲. 让一个摄像头覆盖最多节点.

4. **根节点 corner case / Root might still be uncovered**

    DFS 结束后, **根节点可能返回 0** (两子都 = 1, 自己未覆盖, 又没有父辈兜底). 此时手动 `cameras++`. **必须放在 main 函数里检查**, 不能在 dfs 内做 (dfs 内拿不到"是不是根" 的信息).

5. **为什么是贪心 / Why this is greedy, not DP**

    技术上是"树形 DP", 但**状态只有 3 个**, 转移**确定** (没有"在装/不装之间选 min" 的分支). 每个节点的决策只看子节点状态 — **局部最优 → 全局最优**: "叶子不装, 装父亲" 这一步在每个子树都是省的, 累加起来还是最少.

    与一般树形 DP 的区别: 一般 DP 会维护 `dp[node][0/1/2]` 三个数, 取 min 转移. 这里**直接用状态码代替 DP 数组**, 因为转移是单值确定的.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int cameras = 0;
        // 0 = uncovered, 1 = covered (no camera), 2 = has camera
        int dfs(TreeNode* node) {
            if (!node) return 1;                                   // 空节点视为"已覆盖" — 让叶子上报 0, 父亲装
            int l = dfs(node->left);
            int r = dfs(node->right);
            if (l == 0 || r == 0) { cameras++; return 2; }         // 任一子未覆盖 → 必须装
            if (l == 2 || r == 2) return 1;                        // 任一子有摄像头 → 自己被覆盖
            return 0;                                              // 两子都仅"被覆盖" → 自己未覆盖, 等父亲
        }
        int minCameraCover(TreeNode* root) {
            if (dfs(root) == 0) cameras++;                         // 根没人罩, 自己装
            return cameras;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minCameraCover(self, root: Optional[TreeNode]) -> int:
            self.cameras = 0                                       # 用 self 而非 nonlocal, 避免闭包 boilerplate

            def dfs(node):
                if not node:
                    return 1                                       # 空节点 = 已覆盖
                l, r = dfs(node.left), dfs(node.right)             # 元组解包同时拿左右, Pythonic
                if l == 0 or r == 0:
                    self.cameras += 1
                    return 2
                if l == 2 or r == 2:
                    return 1
                return 0

            if dfs(root) == 0:
                self.cameras += 1
            return self.cameras
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var minCameraCover = function(root) {
        let cameras = 0;
        // 闭包内修改外层 cameras: JS 闭包默认能改, 不像 Python 需要 nonlocal
        const dfs = (node) => {
            if (!node) return 1;
            const l = dfs(node.left);
            const r = dfs(node.right);
            if (l === 0 || r === 0) { cameras++; return 2; }
            if (l === 2 || r === 2) return 1;
            return 0;
        };
        if (dfs(root) === 0) cameras++;
        return cameras;
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问一次.
- **Space**: O(h) — 递归栈深 = 树高. 最坏 O(n).

## 相关题目

- 0337\. House Robber III (待补) — 树形 DP, 每节点维护"偷/不偷" 两个状态
- 0124\. Binary Tree Maximum Path Sum (待补) — 后序 + 自底向上传值的另一个经典
- [0236. Lowest Common Ancestor of a Binary Tree](../../07-binary-tree/0236-lowest-common-ancestor-of-a-binary-tree/README.md) — 同款"后序自底向上汇报状态"
- 0104\. Maximum Depth of Binary Tree (待补) — 后序入门, 子树高度汇报
- 0834\. Sum of Distances in Tree (待补) — 树形 DP 高级题

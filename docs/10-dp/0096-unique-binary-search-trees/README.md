# 0096. Unique Binary Search Trees / 不同的二叉搜索树

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Tree, BST, Math · 动态规划, 树, 二叉搜索树, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/unique-binary-search-trees/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给整数 `n`, 求由 `1..n` 节点能构成多少种**结构不同**的 BST.

**中文**: 给 `n`, 由 `1..n` 节点能构成多少种结构不同的二叉搜索树.

## Key Insights

1. **核心性质: 根定下来 → 左右子树节点集合就确定 / Root fixes the split**

    BST 满足"左 < 根 < 右". 选 `i` 当根:

    - **左子树**: 节点 `1..i-1` (共 `i-1` 个)
    - **右子树**: 节点 `i+1..n` (共 `n-i` 个)

    > 这是 BST 跟普通二叉树最大的区别 — 节点值有序, 选了根, 左右"成员名单" 就锁死, 不用再分配.

2. **DP 状态: dp[n] 只跟"节点个数" 有关, 与节点具体值无关 / Count depends only on size**

    `1..i-1` 共 `i-1` 个节点 → 能形成的 BST 种数等于 `dp[i-1]`. 跟节点是 `1..i-1` 还是 `5..i+3` 没关系, 只看**节点个数**. 这是能用一维 DP 的根本原因.

    > 任何"size 决定形状数" 的问题都能套这个结构.

3. **转移: 枚举根 → 左右乘积求和 / Sum over roots, product over subtrees**

    左右子树独立, 互相组合 → 乘法. 枚举所有 `i` 作根 → 加法.

    $$dp[n] = \sum_{i=1}^{n} dp[i-1] \cdot dp[n-i]$$

    > 跟 [0343 整数拆分](../0343-integer-break/README.md) 同款"枚举切点 + 子问题相乘" 结构, 但 0343 求 max, 这里求 sum.

4. **初始化 `dp[0] = 1` 才能让乘法工作 / Empty tree = 1**

    空子树**算一种结构** (只有"没有节点" 这一种). 若 `dp[0] = 0`, 那任何含空子树的根 (例如 `i = 1`, 左子树为空) 都会被乘成 0, 全盘错.

    > "空集算一种" 跟"空乘 = 1" 一样, 是组合 DP 里很常见的边界处理.

5. **这就是卡特兰数 / Catalan number**

    `dp[n] = C(n)`, 第 n 个 Catalan 数 `C_n = \binom{2n}{n} / (n+1)`. 同序列还覆盖: 合法括号串数, n 对入栈出栈序列数, n+2 边凸多边形三角剖分数, ...

    > 看到 `Σ dp[i] * dp[n-1-i]` 就立刻反应"卡特兰", 这个套路出现频率极高.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numTrees(int n) {
            vector<int> dp(n + 1, 0);
            dp[0] = 1;                                             // 空树算一种, 让乘法 work
            dp[1] = 1;
            for (int i = 2; i <= n; i++) {
                for (int j = 1; j <= i; j++) {                     // 枚举根 j
                    dp[i] += dp[j - 1] * dp[i - j];                // 左 j-1 个 × 右 i-j 个
                }
            }
            return dp[n];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def numTrees(self, n: int) -> int:
            dp = [0] * (n + 1)
            dp[0] = dp[1] = 1
            for i in range(2, n + 1):
                # sum(...) + 生成器 一行算 dp[i]; 等价 C++ 内层 for 累加
                dp[i] = sum(dp[j - 1] * dp[i - j] for j in range(1, i + 1))
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var numTrees = function(n) {
        const dp = new Array(n + 1).fill(0);
        dp[0] = 1;
        dp[1] = 1;
        for (let i = 2; i <= n; i++) {
            for (let j = 1; j <= i; j++) {
                dp[i] += dp[j - 1] * dp[i - j];
            }
        }
        return dp[n];
    };
    ```

## Complexity

- **Time**: O(n²) — 双层循环.
- **Space**: O(n).

## 相关题目

- [0343. Integer Break](../0343-integer-break/README.md) — 同"枚举切点" 结构, 但求 max 而非 sum
- 0095\. Unique Binary Search Trees II (待补) — 不仅算个数, 要返回所有 BST (回溯 + 同款分治)
- 1259\. Handshakes That Don't Cross (待补) — 同卡特兰数
- 0022\. Generate Parentheses (待补) — 合法括号串, 也是卡特兰

# 0437. Path Sum III / 路径总和 III

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Prefix Sum, Hash Map, Backtracking · 二叉树, DFS, 前缀和, 哈希表, 回溯
    - **Link**: [LeetCode](https://leetcode.com/problems/path-sum-iii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count downward paths (any node → any descendant) summing to target** → **prefix sum on root-to-current-node path + hash count** ([0560](../../01-array/0560-subarray-sum-equals-k/README.md) on a tree). **Backtrack**: `++cnt[cur]` before recursing, `--cnt[cur]` after — otherwise sibling subtrees pollute each other.
>
> **中文**: **数向下路径 (任一祖先 → 任一后代) 和 = target 的个数** → **根到当前节点路径上的前缀和 + hash count** (0560 搬到树上). **回溯**: 进入 ++, 离开 -- — 否则兄弟子树互相污染.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 二叉树. 求**路径和 = targetSum** 的**向下**路径数. 路径**必须父→子**方向 (不能跨兄弟), 但**不必从 root 开始, 也不必到 leaf 结束**. 节点值可**负**.

**中文**: 数向下路径和 = target 的个数, 任意起终点.

## Key Insights

1. **🔑 灵魂: 把 [0560](../../01-array/0560-subarray-sum-equals-k/README.md) 搬到树上 / Reduce to 0560 on a tree**

    数组 0560 求"子数组和 == k" 的个数: 前缀和 + hash count, 一次扫 O(n).

    **本题的路径 = "从某祖先到当前节点" 的子路径** ⇔ 数组里"从某位置到当前位置" 的子数组.

    → **完全同构**: 沿**根到当前节点** 的路径记 prefix sum, 当前 `cur - target` 在 hash 里的次数就是"以某祖先为起点的合法路径数". 累加即答案.

    > **树上的 prefix + hash** = 数组 0560 的抬升版. 灵魂**完全一致**, 只是"顺序遍历" 变成"树 DFS".

2. **🔑 灵魂 backtrack: 进入 ++, 离开 -- / Backtracking is mandatory**

    Yang 的关键两行:

    ```cpp
    ++cnt[cur];                    // 进入: cur 加入路径
    res += dfs(node->left,  cur);
    res += dfs(node->right, cur);
    --cnt[cur];                    // 离开: cur 从路径移除
    ```

    **为啥必须 --?** 路径只允许**祖先 → 后代**, 不能**跨兄弟**. 若不 --, 右子树会看到左子树的 prefix, 误判"某左子树节点到右子树节点的路径" (**跨兄弟**, 违规).

    ```
    示范: root = 1, left = 2, right = 3, target = 3
    进入 root: cnt = {0:1, 1:1}
    进入 left (2): cur=3, hash 命中 cnt[0]=1 → res+=1 (root→left 和 3 ✓)
      cnt = {0:1, 1:1, 3:1}
    离开 left, 若**不 --**: cnt 仍含 3:1
    进入 right (3): cur = 4, 查 cnt[1]=1 → res+=1 (root→right 和 4? 不对, target 是 3)
                       查 cnt[3]=1 → 误认为"跨过 2 到 3" 有路径
    ```

    → **backtrack 的 --** 是"树上路径不能跨兄弟" 的显式保证. 少一行就 WA.

    > **回溯的本质**: **进入子问题前修改状态, 离开时 undo**. 保证子问题之间**独立**. 跟 [0051 N-Queens](../../08-backtracking/0051-n-queens/README.md) 的 board 撤回同源思想.

3. **🔑 `cnt[0] = 1` 哨兵 / Sentinel: empty prefix**

    - 语义: **空前缀** 在"root 之前", prefix sum 是 0.
    - 若从 root 开始到当前节点和 == target, `cur - target = 0` 命中哨兵 → 正确计数.
    - 少这行, 从 root 开始的路径全部漏.

    > 同 [0560](../../01-array/0560-subarray-sum-equals-k/README.md) / [0523](../../01-array/0523-continuous-subarray-sum/README.md) / [0974](../../01-array/0974-subarray-sums-divisible-by-k/README.md) 家族的哨兵.

4. **🔑 `long long` 防溢出 / long long overflow guard**

    LC constraints: 节点值 `[-10^9, 10^9]`, target `[-1000, 1000]`, 深度可达 1000. **prefix 累加可能 ±10^12**, 远超 int (2×10^9). **必须 long long**.

    Yang 用 `long long cur` + `unordered_map<long long, int>` — 关键防御.

    > 数值题看到"大 value + 累加" 立刻查溢出. **本题严格必需 long long**.

5. **🔑 朴素 O(n²) vs prefix O(n) / Naive vs prefix**

    | 方法 | Time | Space | 思路 |
    |---|---|---|---|
    | 双 DFS | O(n²) | O(h) | 每个节点作起点跑一遍 "以此为起点向下的路径" |
    | **prefix + hash + backtrack** | **O(n)** | O(h) | 一遍 DFS + hash count |

    - 双 DFS: 外层 DFS 每个节点作起点 (n 次), 内层 DFS 求以它为起点的路径 (最坏 O(n)) → **O(n²)**.
    - **prefix + hash**: 一次 DFS 全解决, hash 里始终只有"根到当前" 这条链的 prefix, O(1) 查询. 天壤之别.

    > **"每个节点作起点独立 DFS" 是 O(n²) 陷阱**. 见到就反射 prefix + hash 优化.

6. **🔑 跟 [0112 Path Sum](../0112-path-sum/README.md) / 0113 家族的对比 / vs path-sum family**

    | 题 | 路径类型 | 求 | 方法 |
    |---|---|---|---|
    | [0112 Path Sum](../0112-path-sum/README.md) | root → leaf | 是否存在和 = target | DFS 短路返 bool |
    | 0113 Path Sum II (待补) | root → leaf | 所有和 = target 的路径 | DFS 回溯收结果 |
    | **0437 (本题)** | **任一祖先 → 任一后代** | **个数** | **prefix + hash + backtrack** |
    | [0124 Binary Tree Max Path Sum](待补) | 任意两节点 | 最大路径和 | 后序返子树最优 + 全局 max |

    > **一族多题**, 路径定义 (root-leaf vs 任意) + 目标 (存在/全部/个数/最大) 组合出不同技巧.

7. **🔑 复杂度 O(n) 时间, O(h) 空间 / Linear + height**

    - Time: 每节点访问 1 次, hash O(1).
    - Space: 递归栈 O(h) + hash 最多 h 项 (根到当前链的 prefix).

## Solution

=== "C++"
    ```cpp
    class Solution {
        unordered_map<long long, int> cnt;
        int target;

        int dfs(TreeNode* node, long long cur) {
            if (!node) return 0;
            cur += node->val;
            int res = 0;
            auto it = cnt.find(cur - target);
            if (it != cnt.end()) res += it->second;                  // 有几条祖先到我的路径 = target
            ++cnt[cur];                                              // 进入: cur 加入 hash
            res += dfs(node->left,  cur);
            res += dfs(node->right, cur);
            --cnt[cur];                                              // 离开: 回溯 undo
            return res;
        }
    public:
        int pathSum(TreeNode* root, int targetSum) {
            target = targetSum;
            cnt[0] = 1;                                              // 哨兵: 空前缀
            return dfs(root, 0);
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def pathSum(self, root, targetSum: int) -> int:
            cnt = defaultdict(int)
            cnt[0] = 1              # 哨兵

            def dfs(node, cur):
                if not node: return 0
                cur += node.val
                res = cnt[cur - targetSum]  # 之前有几个 prefix = cur - target
                cnt[cur] += 1                # 进入
                res += dfs(node.left, cur)
                res += dfs(node.right, cur)
                cnt[cur] -= 1                # 离开: backtrack
                return res

            return dfs(root, 0)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} targetSum
     * @return {number}
     */
    var pathSum = function(root, targetSum) {
        const cnt = new Map();
        cnt.set(0, 1);
        const dfs = (node, cur) => {
            if (!node) return 0;
            cur += node.val;
            let res = cnt.get(cur - targetSum) || 0;
            cnt.set(cur, (cnt.get(cur) || 0) + 1);
            res += dfs(node.left, cur);
            res += dfs(node.right, cur);
            cnt.set(cur, cnt.get(cur) - 1);         // backtrack
            return res;
        };
        return dfs(root, 0);
    };
    ```

## Complexity

- **Time**: O(n) — 每节点访问 1 次.
- **Space**: O(h) — 递归栈 + hash 最多 h 项.

## 相关题目

- [0560. Subarray Sum Equals K](../../01-array/0560-subarray-sum-equals-k/README.md) — **数组版本, 本题的思想母题**
- [0112. Path Sum](../0112-path-sum/README.md) — 判 root → leaf 是否存在 target
- [0523. Continuous Subarray Sum](../../01-array/0523-continuous-subarray-sum/README.md) — 前缀 mod
- [0974. Subarray Sums Divisible by K](../../01-array/0974-subarray-sums-divisible-by-k/README.md) — 前缀 mod count
- [0525. Contiguous Array](../../01-array/0525-contiguous-array/README.md) — mapping trick
- [1248. Count Number of Nice Subarrays](../../01-array/1248-count-number-of-nice-subarrays/README.md) — mapping + 0560
- 0113\. Path Sum II (待补) — root → leaf 所有 target 路径
- 0124\. Binary Tree Maximum Path Sum (待补) — Hard, 任意两节点最大和
- 0129\. Sum Root to Leaf Numbers (待补) — root → leaf 数字之和
- 0666\. Path Sum IV (待补) — 编码树 + 路径和
- 0988\. Smallest String Starting From Leaf (待补) — 叶到根字典序

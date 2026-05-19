# 0216. Combination Sum III / 组合总和 III

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, Combinations · 回溯, 组合
    - **Link**: [LeetCode](https://leetcode.com/problems/combination-sum-iii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Return all combinations of `k` **distinct** numbers from `[1, 9]` whose sum is `n`. Each number used at most once.

**中文**: 从 `[1, 9]` 里挑 `k` 个**不重复**的数, 使它们的和恰好等于 `n`. 返回所有这样的组合.

## 思路

### Core idea

**[0077](../0077-combinations/README.md) + 和等于 n 的约束**. 同款回溯三件套 (push → recurse → pop) + `startIndex` 防重, 只是:

1. 候选范围固定 `[1, 9]` (不是 `[1, n]`).
2. 终止条件从"path.size() == k" 加一道二审 — `path.size() == k && curSum == n` 才收果实.
3. 用一个 **`curSum` 参数**累计当前和, **隐式回溯** (int 按值传, 无需手动撤销).

### Key Insights

1. **跟 [0077](../0077-combinations/README.md) 的差异: 候选 + 终止 / Two tweaks on 0077**

    | 项 | 0077 | 0216 |
    |---|---|---|
    | 候选范围 | `[1, n]` | `[1, 9]` 固定 |
    | 终止条件 | `path.size() == k` 必收 | `path.size() == k` **且** `curSum == n` 才收 |
    | 路径状态 | 只有 `path` + `startIndex` | 加 `curSum` |

    其它一切骨架完全相同. 这就是回溯系列的快感: **换一两个 hook**.

2. **`curSum` 用参数传 = 隐式回溯 / Pass curSum, no manual restore**

    `int curSum` 按值传, 同 [0129](../../07-binary-tree/0129-sum-root-to-leaf-numbers/README.md) / [0113 的 targetSum](../../07-binary-tree/0113-path-sum-ii/README.md): 父函数自己的 curSum 永远不会被子调用动. 不需要 `curSum -= i` 撤销.

    **如果**把 curSum 升成类成员/全局, 就必须手动 `curSum -= i` 撤销 — 看个人喜好, Yang 这版的参数传递更干净.

3. **能加的剪枝 (Yang 这版没写, 但回溯题面试常被问)**

    - **范围剪枝** (照搬 0077): `i <= 9 - (k - path.size()) + 1`. 剩下不够凑齐 k 个就停.
    - **和-超出剪枝**: 因为所有数都 ≥ 1, `curSum > n` 之后再往下加只会更大, 直接 return. 在 `path.size() == k` 检查之前加 `if (curSum > n) return;` 一行即可.
    - **和-不足剪枝 (高级)**: 算一下"剩下能加的最大可能值"是否够到 n. 比较烦, 一般不写.

    LC 数据小 ([1,9] 只有 9 个数), 不剪也快, **但面试遇到必须能说出范围剪枝 + 超出剪枝两条**.

4. **每个数最多用一次 → `i + 1`, 不是 `i` / Each number used at most once**

    下一层 `backtracking(k, n, i + 1, ...)` 用 `i + 1` 表示"下次只能从更大的数开始挑". 写成 `i` 就允许重复挑同一个数, 那是 0039 Combination Sum (待补) 的题目.

5. **回溯 vs DP / Backtracking vs DP**

    这题"组合数和等于 n" 听起来像背包. 但题目要求**列出所有具体组合**, 不是只问方案数 → 回溯才能给出每条路径. DP 适合"有多少种组合数加和等于 n" 这类只问数量的题. 听到"返回所有" 几乎一定回溯, 听到"返回数量" 就考虑 DP.

### 一句话总结

**0077 + 和等于 n. 同回溯三件套, `curSum` 按值传参实现隐式回溯, `[1, 9]` 范围, `path.size() == k && curSum == n` 才收.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> res;
        vector<int> path;
        void backtracking(int k, int n, int startIndex, int curSum) {
            if ((int)path.size() == k) {
                if (curSum == n) res.push_back(path);    // 二审: 既要满 k 个又要和等 n
                return;
            }
            // 可加剪枝: if (curSum > n) return;
            // 可加剪枝: 上界改成 9 - (k - path.size()) + 1
            for (int i = startIndex; i <= 9; i++) {
                path.push_back(i);
                backtracking(k, n, i + 1, curSum + i);   // curSum + i 按值传 → 隐式回溯
                path.pop_back();
            }
        }
        vector<vector<int>> combinationSum3(int k, int n) {
            backtracking(k, n, 1, 0);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def combinationSum3(self, k: int, n: int) -> list[list[int]]:
            res, path = [], []
            def backtrack(start: int, cur_sum: int):
                if len(path) == k:
                    if cur_sum == n:
                        res.append(path[:])              # 快照拷贝, 同 0077
                    return
                # 可加剪枝: if cur_sum > n: return
                for i in range(start, 10):               # range 右开, [start, 9]
                    path.append(i)
                    backtrack(i + 1, cur_sum + i)        # cur_sum + i 按值传, 隐式回溯
                    path.pop()
            backtrack(1, 0)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} k
     * @param {number} n
     * @return {number[][]}
     */
    var combinationSum3 = function(k, n) {
        const res = [], path = [];
        const backtrack = (start, curSum) => {
            if (path.length === k) {
                if (curSum === n) res.push([...path]);   // 扩展运算符浅拷贝
                return;
            }
            for (let i = start; i <= 9; i++) {
                path.push(i);
                backtrack(i + 1, curSum + i);            // number 按值传, 隐式回溯
                path.pop();
            }
        };
        backtrack(1, 0);
        return res;
    };
    ```

## Complexity

- **Time**: O(C(9, k) × k) — 最多枚举 C(9,k) 条路径, 每条拷贝 O(k). k ≤ 9 实际很小.
- **Space**: O(k) recursion + path. 输出本身 O(答案数 × k).

## 易错点

- **`i + 1` 不是 `i`**: 同一个数不能用两次. 写成 `i` 就允许 `[2, 2, 5]` 这种, 那是 0039 题.
- **range 是 `[1, 9]`, 不是 `[1, n]`**: 跟 0077 的范围不一样, 别照抄. n 是目标和, 不是范围上界.
- **`curSum` 别在终止前判**: 要先检查 `path.size() == k`, 再判 sum. 顺序反了会丢答案 (path 不满 k 个但 sum 已等于 n 时, 应该继续找而不是收).
- **没加剪枝在大数据上没事**: LC 这题数据范围小. 但**面试**要主动提出至少范围剪枝 + 超出剪枝两条.
- **`curSum` 类成员化要记得手动撤销**: 如果不用参数传, 升成类成员, 那进入 push 后要 `curSum += i`, pop 之后 `curSum -= i`. Yang 这版用参数传所以完全省掉这一步.

## 相关题目

- [0077. Combinations](../0077-combinations/README.md) — 直接前置题, 本题在它上面加 sum 约束
- [0113. Path Sum II](../../07-binary-tree/0113-path-sum-ii/README.md) — 同款"路径 + 和"约束, 但走的是真二叉树
- [0129. Sum Root to Leaf Numbers](../../07-binary-tree/0129-sum-root-to-leaf-numbers/README.md) — 同款"累加参数传值" 隐式回溯
- 0039. Combination Sum (待补) — 允许重复选同一数, `recurse(i)` 不是 `recurse(i+1)`
- 0040. Combination Sum II (待补) — 输入有重复, 需要同层去重
- 0017. Letter Combinations of a Phone Number (待补) — 不同集合每位选一个的回溯入门

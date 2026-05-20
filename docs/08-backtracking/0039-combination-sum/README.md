# 0039. Combination Sum / 组合总和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, Array · 回溯, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/combination-sum/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given distinct positive `candidates` and a positive `target`, return all unique combinations where the chosen numbers sum to `target`. **Each candidate may be used unlimited times.**

**中文**: 给一组**互不相同**的正整数 `candidates` 和正整数 `target`, 返回所有和为 `target` 的不同组合. **每个数可以无限次使用**.

## 思路

### Core idea

回溯三件套不变, **唯一一行差异**: 下一层递归传 `i` 而不是 `i + 1`. 这样同一个元素在同一条 path 里可以被再次挑出来. `startIndex` 仍然在 — 它防的不是"重复用同一个数", 而是"换顺序产生的重复组合".

### Key Insights

1. **唯一改动: `recurse(i)` 不是 `recurse(i + 1)` / The defining one-liner**

    | 题 | 下一层起点 | 同元素能否重复 |
    |---|---|---|
    | [0077](../0077-combinations/README.md) / [0216](../0216-combination-sum-iii/README.md) | `i + 1` | ❌ 一次性 |
    | **0039 (本题)** | `i` | ✅ 无限次 |
    | 0040 (待补) | `i + 1`, 但同层跳重值 | 每个**位置**一次, 但 candidates 有重复值 |

    回溯系列的"允许重复"和"防顺序重复"是两条独立的轴 — 别混淆.

2. **`startIndex` 仍然必须 / Still need startIndex for order-dedup**

    可能直觉觉得"既然允许重复, 还要 startIndex 干嘛". 错. startIndex 防的是**组合的顺序**: `[2,2,3]` 跟 `[3,2,2]` 是同一种组合 (题目要的是无序集合). startIndex 强制"先选小的, 再选大的" → 顺序唯一.

    去掉 startIndex 的话: `recurse(0)` 每次都从头, 那 `[2,3]` 跟 `[3,2]` 会都生成出来 — 这是 0046 排列的逻辑, 不是组合.

3. **早停剪枝: `curSum >= target` / Overshoot prune**

    candidates 都是正整数, sum 单调增. 一旦 `curSum > target`, 再往下加只会更大 → 直接 return. Yang 的版本里 `if (curSum >= target)` 一句把"找到 (== target) 收果实" 和 "超过 (> target) 剪掉" 合并写, 简洁.

4. **更紧的剪枝: sort + break / Sort once, break inside loop**

    进阶版: 入口先 `sort(candidates)`, 然后 for 循环里:
    ```cpp
    if (curSum + candidates[i] > target) break;  // 不是 continue!
    ```
    因为有序, 当前 i 都超了, 后面更大的 i 全超 → 整个循环可以 break. 减少**同层**多余的 push/pop. Yang 这版没做, 数据小不剪也过.

5. **隐式回溯 + 显式回溯并存 / Both backtrackings on display**

    - `curSum + candidates[i]` 按值传 → curSum 在父调用里不变, **隐式回溯** (跟 [0129](../../07-binary-tree/0129-sum-root-to-leaf-numbers/README.md) / [0216](../0216-combination-sum-iii/README.md) 同款).
    - `path` 共享 + `push_back/pop_back` → **显式回溯** (跟 [0113](../../07-binary-tree/0113-path-sum-ii/README.md) / [0077](../0077-combinations/README.md) 同款).

    一个函数里两种回溯方式自然共存 — 每个状态量挑最合适的那种.

6. **复杂度难写紧 / Complexity is loose**

    因为允许重复, 答案树宽度 ~ n, 深度 ~ target/min(candidates). 最坏 O(n^(target/min) × target). 数据小不需要担心 (LC target ≤ 40, candidates ≥ 2).

### 一句话总结

**0077 + 允许同元素重复 = `recurse(i)`. `startIndex` 仍要 (防顺序重复). `curSum > target` 早停, `curSum == target` 收果实.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> res;
        vector<int> path;
        void backtracking(const vector<int>& candidates, int target, int curSum, int startIndex) {
            if (curSum >= target) {
                if (curSum == target) res.push_back(path);
                return;                                   // > target 时也在这里剪掉
            }
            for (int i = startIndex; i < (int)candidates.size(); i++) {
                path.push_back(candidates[i]);
                backtracking(candidates, target, curSum + candidates[i], i);  // i 不是 i+1: 允许重复
                path.pop_back();
            }
        }
        vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
            backtracking(candidates, target, 0, 0);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
            res, path = [], []
            def backtrack(start: int, cur_sum: int):
                if cur_sum >= target:
                    if cur_sum == target:
                        res.append(path[:])               # 切片拷贝快照
                    return
                # enumerate 从 start 开始, 同时拿 i 和 candidates[i]
                # 等价 C++ for (i = start; i < n; ++i) { path.push_back(candidates[i]); ... }
                for i in range(start, len(candidates)):
                    path.append(candidates[i])
                    backtrack(i, cur_sum + candidates[i]) # i 而非 i+1: 允许重复
                    path.pop()
            backtrack(0, 0)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} candidates
     * @param {number} target
     * @return {number[][]}
     */
    var combinationSum = function(candidates, target) {
        const res = [], path = [];
        const backtrack = (start, curSum) => {
            if (curSum >= target) {
                if (curSum === target) res.push([...path]);  // 扩展运算符浅拷贝
                return;
            }
            for (let i = start; i < candidates.length; i++) {
                path.push(candidates[i]);
                backtrack(i, curSum + candidates[i]);        // i 而非 i+1: 允许重复
                path.pop();
            }
        };
        backtrack(0, 0);
        return res;
    };
    ```

### 进阶: sort + break 剪枝

```cpp
sort(candidates.begin(), candidates.end());
// 然后在 for 里:
if (curSum + candidates[i] > target) break;
path.push_back(candidates[i]);
backtracking(candidates, target, curSum + candidates[i], i);
path.pop_back();
```

排序成本 O(n log n) 一次性, 但每个分支可以 break 整个循环而不是单独走 if. 数据大时差距明显.

## Complexity

- **Time**: O(n^(target/min) × target) 上界, 实际通常远低于此 (剪枝 + 有效组合数).
- **Space**: O(target/min) recursion depth + path.

## 易错点

- **`recurse(i)` 不是 `recurse(i + 1)`**: 这就是这题与 0077/0216 的唯一区别. 写错就变成"每个数只能用一次", 答案大量缺失.
- **删掉 `startIndex` 会变成排列**: 不能因为"允许重复"就以为不需要 startIndex 了. 见 Insight 2.
- **`curSum > target` 必须剪**: 不剪也对但极慢, 因为重复选会让 sum 无限增长. `curSum >= target` 收口里包含了这个剪枝.
- **`candidates` 都是正整数才能这样剪**: 题目保证. 如果有 0 或负数, `curSum` 不再单调 → 不能用 sum 早停, 而且 0 会让 `recurse(i)` 死循环.
- **`(int)candidates.size()`**: C++ 里 `size()` 是 `size_t` (unsigned), 比较 int 时有坑. Yang 这版没在风险点比较, 但 cast 一下是好习惯.
- **同元素的多次使用算"组合的不同表示" 吗**: 不算. `[2,2,3]` 是一种组合 (无序 multiset). 我们的去重已经在 startIndex 强制顺序里包了, 不需要额外比较.

## 相关题目

- [0077. Combinations](../0077-combinations/README.md) — 直接前置, 本题改 `recurse(i+1) → recurse(i)`
- [0216. Combination Sum III](../0216-combination-sum-iii/README.md) — 同款 sum 约束, 但不允许重复 + 范围固定 [1,9]
- [0017. Letter Combinations of a Phone Number](../0017-letter-combinations-of-a-phone-number/README.md) — 同回溯模板, 不同形态 (多集合每位一选)
- [0040. Combination Sum II](../0040-combination-sum-ii/README.md) — candidates 有重复值, 需要**同层去重**, 是这题的"反向"难点
- 0377. Combination Sum IV (待补) — 求**数量**不求列表 → DP 才合适, 不要再回溯
- 0322. Coin Change (待补) — 同款"无限次重复 + 凑 target", 但求最少数量 → DP

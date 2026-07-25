# 0546. Remove Boxes / 移除盒子

!!! info "Meta"
    - **Difficulty**: Hard (声称的 Hard×2 之一)
    - **Tags**: DP, 3D Interval DP, Memoization · 动态规划, 三维区间 DP, 记忆化
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-boxes/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Remove contiguous same-color runs (score = len²), maximize total** → **3D interval DP `dp[i][j][k]`** where **k = # same-color boxes "hanging" on the left of `i`** (waiting to merge with `box[i]`); two choices per state: **remove now** (`(k+1)² + dfs(i+1, j, 0)`) or **wait for merge** (find m with `box[m] == box[i]`, `dfs(i+1, m-1, 0) + dfs(m, j, k+1)`).
>
> **中文**: **消同色连段拿 len² 分, 求总分最大** → **3D 区间 DP `dp[i][j][k]`**, **k = i 左边悬挂等待合并的同色盒数**; 每步二选: **立即消** ((k+1)² + 剩余) 或 **等合并** (先消中间, 再让 box[m] 跟 box[i] 一起消, k+1 传递下去).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 一排彩色盒子. 每次可**消掉一段连续同色**, 得分 = **段长²**. 消掉后**两边收拢** — 可能新形成同色连段. 求**最大总分**.

- 例: `[1,3,2,2,2,3,4,3,1]` → 23. 消 3 个 2 (9), 消 3 (1), 消 4 (1), 剩 `[1,3,2 → 消 3 → 消 4 → ...]` — 具体解释见 LC.

**中文**: 消同色段, 每次得 len² 分, 求最大分. **消掉后两边合并** 是难点.

## Key Insights

1. **🔑 灵魂难点: 消掉后两边合并 / The killer twist: post-removal merges**

    ```
    [1, 3, 2, 2, 3, 1]
    消 3 (中间, 1 分) → [1, 2, 2, 1]
    消 2, 2 (4 分) → [1, 1]
    消 1, 1 (4 分). 总 9.
    ```

    → **消 3 的动机**是**打通** 后续更长的同色 — 这种"**为未来铺路**" 让**朴素区间 DP `dp[i][j]`失效**.

    > 传统 dp[i][j] 无法表达"这段的右邻居和别处能合并". 需要**额外维度记录合并意图**.

2. **🔑 神招: 3D 状态 — `dp[i][j][k]` / Genius 3D state**

    - `[i, j]`: 当前处理的连续子数组.
    - **`k`**: **`i` 左边"悬挂"的同色盒数** — 意思是"外面还有 k 个跟 `box[i]` 同色的盒子等着跟 `box[i]` 一起消".

    **答案 = `dp[0][n-1][0]`** — 一开始外面没悬挂.

    > **"额外维度承载合并意图"** 是这题的天才. `k` 让"我等等再一起消" 这种意图可以传递到子问题.

3. **🔑 两种决策 / Two choices**

    **决策 A — 立即消**:

    ```cpp
    (k + 1) * (k + 1) + dfs(i + 1, j, 0)
    ```

    - 把 box[i] 和左边悬挂的 k 个**一起消掉** → **(k+1)²** 分.
    - 剩余 [i+1, j], **无悬挂** (k = 0).

    **决策 B — 等合并** (对 j > m > i, `box[m] == box[i]`):

    ```cpp
    dfs(i + 1, m - 1, 0) + dfs(m, j, k + 1)
    ```

    - **先消中间 [i+1, m-1]** (无悬挂, 0).
    - **然后 box[m] 跟 box[i] + k 个合并** → 相当于 dfs(m, j, k+1) — **k+1 是把 box[i] 加入悬挂**.

    → **取两者最大**.

    > **"先消中间, 让远端相同色合并" 就是穿越时空的合并**. Yang 这几行代码是 DP 里的诗.

4. **🔑 预压缩: 跳过连续同色 / Pre-compress consecutive same colors**

    Yang 的一段:

    ```cpp
    int i0 = i, k0 = k;
    while (i < j && box[i + 1] == box[i]) { i++; k++; }
    ```

    若 box[i], box[i+1], box[i+2] 都同色, 直接把它们**并入悬挂** — `k += 相同色个数`. 简化后续 DP 分支.

    - `i0, k0` 存**原始参数** 用于 memo (下面 `dp[i0][j][k0]`).

    > **区间 DP 里"预压缩相邻同色" 是重要优化** — 跟 [0664 Strange Printer](../0664-strange-printer/README.md) 的预处理同源.

5. **🔑 memo 键必须用**原始** (i0, k0) / Memo key with pre-compression indices**

    ```cpp
    return dp[i0][j][k0] = res;
    ```

    - 状态是**调用时** 的 (i, k), 不是压缩后的.
    - 若用压缩后的 (i, k), 会**丢失原状态映射**.

    > 优化改了参数 → memo 必须用**原参数** 存. 记住这个"压缩 + memo 分离" 模式.

6. **🔑 3D DP 需要**3D 数组** / 3D array required**

    Yang 直接开 `int dp[100][100][100]` (全局静态零初始化). 因为 constraint n ≤ 100.

    - **`dp[i][j][k] == 0`** 表示 "未算" (因为答案 ≥ 1 若非空).
    - Hash map 也行 `unordered_map<tuple<int,int,int>, int>` 但**慢一档** (hash 常数).

    > **"用 0 作 未算 flag"** 是内存优化, 只适用于**保证答案 > 0** 的情况. 若 0 是合法答案, 得改 -1 或用 optional/bool marker.

7. **🔑 复杂度 O(n⁴) 时间, O(n³) 空间 / Notorious complexity**

    - 状态数: n × n × n = O(n³).
    - 每状态试 O(n) 个 m.
    - Total: **O(n⁴)** — n = 100 时 10⁸, 险过.

    > **LC Hard 里最贵的复杂度之一**. 常数优化 (memo, 压缩) 关键.

8. **🔑 跟 [0664 Strange Printer](../0664-strange-printer/README.md) 的对比 / vs 0664**

    | | 0664 | **0546 (本题)** |
    |---|---|---|
    | 动作 | 打印 (覆盖) | 消除 (合并) |
    | 状态 | 2D `dp[i][j]` | **3D `dp[i][j][k]`** |
    | 得分 | 打印数 (min) | 段长² (max) |
    | 合并方向 | 打 t[i] 顺路打 t[k] | 消中间让 box[m] 归入 |

    > 同族区间 DP, 但 0546 因为**"消掉后段长变化"** 需要额外维度. **是 0664 的进阶变体**.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int dp[100][100][100] = {};                          // 全局零初始化
        vector<int> box;

        int dfs(int i, int j, int k) {
            if (i > j) return 0;
            if (dp[i][j][k]) return dp[i][j][k];             // memo (0 = 未算, 因为答案 > 0)

            int i0 = i, k0 = k;
            // 预压缩: 把 box[i] 后连续同色并入悬挂
            while (i < j && box[i + 1] == box[i]) { i++; k++; }

            // 决策 A: 立即消 (k+1)²
            int res = (k + 1) * (k + 1) + dfs(i + 1, j, 0);

            // 决策 B: 找 m 让 box[m] == box[i], 先消中间, box[m] 合并 (k+1)
            for (int m = i + 1; m <= j; m++) {
                if (box[m] == box[i]) {
                    res = max(res, dfs(i + 1, m - 1, 0) + dfs(m, j, k + 1));
                }
            }
            return dp[i0][j][k0] = res;                      // memo 用原始 (i0, k0)
        }

        int removeBoxes(vector<int>& boxes) {
            box = boxes;
            return dfs(0, boxes.size() - 1, 0);
        }
    };
    ```

=== "Python"
    ```python
    from functools import lru_cache

    class Solution:
        def removeBoxes(self, boxes: list[int]) -> int:
            n = len(boxes)

            @lru_cache(maxsize=None)     # 一装饰器搞定 memo — 3D 元组当 key
            def dfs(i: int, j: int, k: int) -> int:
                if i > j: return 0
                # 预压缩
                while i < j and boxes[i + 1] == boxes[i]:
                    i += 1
                    k += 1
                # 决策 A: 立即消
                res = (k + 1) ** 2 + dfs(i + 1, j, 0)
                # 决策 B: 找 m 合并
                for m in range(i + 1, j + 1):
                    if boxes[m] == boxes[i]:
                        res = max(res, dfs(i + 1, m - 1, 0) + dfs(m, j, k + 1))
                return res

            return dfs(0, n - 1, 0)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} boxes
     * @return {number}
     */
    var removeBoxes = function(boxes) {
        const n = boxes.length;
        // 3D 数组用嵌套 Array.from
        const dp = Array.from({length: n}, () =>
            Array.from({length: n}, () => new Array(n + 1).fill(0)));

        const dfs = (i, j, k) => {
            if (i > j) return 0;
            if (dp[i][j][k]) return dp[i][j][k];
            const i0 = i, k0 = k;
            while (i < j && boxes[i + 1] === boxes[i]) { i++; k++; }
            let res = (k + 1) * (k + 1) + dfs(i + 1, j, 0);
            for (let m = i + 1; m <= j; m++) {
                if (boxes[m] === boxes[i]) {
                    res = Math.max(res, dfs(i + 1, m - 1, 0) + dfs(m, j, k + 1));
                }
            }
            dp[i0][j][k0] = res;
            return res;
        };
        return dfs(0, n - 1, 0);
    };
    ```

## Complexity

- **Time**: O(n⁴) — 状态 n³ × 每状态试 n 个 m.
- **Space**: O(n³) — 3D memo.

## 相关题目

- [0664. Strange Printer](../0664-strange-printer/README.md) — 2D 区间 DP + 合并同色 (本题母题)
- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP + 最后戳
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 区间 DP
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 判定
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 3D 区间 DP (另一种)
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 区间 + 排序
- 1246\. Palindrome Removal (待补) — 区间 DP + 回文合并
- 0087\. Scramble String (待补) — 区间 DP + memo

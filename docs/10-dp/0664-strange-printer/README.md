# 0664. Strange Printer / 奇怪的打印机

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Interval DP, String · 动态规划, 区间 DP, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/strange-printer/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min prints — each print lays one char over a contiguous segment** → **interval DP** `dp[i][j]` = min prints for `t[i..j]`; default `dp[i+1][j] + 1`; **if `t[k] == t[i]`** for some k > i, **merge**: print `t[i]` also covers `t[k]` → `dp[i+1][k-1] + dp[k][j]`. Preprocess: **collapse consecutive dupes**.
>
> **中文**: **最少打印数, 每次打一段连续同字符** → **区间 DP** `dp[i][j]`; 默认 `dp[i+1][j] + 1`; 若 `t[k] == t[i]`, **合并**: 打 `t[i]` 时顺路把 `t[k]` 也打了 → `dp[i+1][k-1] + dp[k][j]`. 预处理**去连续重复**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 打印机每次只能打**一段连续同字符**, 后来的**覆盖** 之前的. 求打出目标串 `s` 的**最少打印次数**.

- 例: `"aaabbb"` → 2 (先打 3 个 a, 再打 3 个 b).
- 例: `"aba"` → 2 (先打 "aaa", 再中间盖 "b"; 或先 b 再 aa; 但不能 1 次).

**中文**: 打印机最少几次能打出 s (每次连续同字符, 可覆盖).

## Key Insights

1. **🔑 预处理: 去连续重复 / Preprocess: collapse duplicates**

    ```
    "aaabbbaa" → "abab"
    ```

    相邻同字符**一次就能打** — 不影响答案. 简化后**长度更短**, DP 更快.

    ```cpp
    string t;
    for (char c : s) if (t.empty() || c != t.back()) t.push_back(c);
    ```

    > **压缩输入是很多字符串 DP 的隐藏优化**. 面试提一句加分.

2. **🔑 灵魂定义: `dp[i][j]` = 打印 `t[i..j]` 的最少次数 / dp[i][j] = min prints for [i..j]**

    典型**区间 DP**: 大区间从**小区间**推出. **base**: `dp[i][i] = 1` (单字符 1 次).

    > **区间 DP 三步曲**: ① 定义区间, ② `len` 从小到大枚举, ③ 每个区间试所有"分割点 / 关键位".

3. **🔑 转移一 (默认): 第 i 字符单独打 / Default: print t[i] alone**

    ```cpp
    dp[i][j] = dp[i + 1][j] + 1;         // 打 t[i] 一次, 剩下 t[i+1..j] 递归
    ```

    这是**上界** — 单独打 t[i] 一定可行, 但可能不最优.

4. **🔑 转移二 (灵魂优化): 合并同色打印 / Merge same-char prints**

    若存在 `k > i` 使 `t[k] == t[i]`:

    - 打 t[i] 时**可以覆盖到 t[k] 的位置** (打一整段, 后面被别的覆盖但**留下 t[k] 那格**).
    - **等价**: t[i] 和 t[k] **共用一次打印**.

    ```
    dp[i][j] = min(dp[i][j], dp[i+1][k-1] + dp[k][j])
    ```

    - **`dp[i+1][k-1]`**: 中间段 t[i+1..k-1] 单独打.
    - **`dp[k][j]`**: 从 k 起, **t[k] 那次打印跟 t[i] 合并** — 但公式**没减 1** 因为**内部还是原样计算**, 减 1 通过 `dp[i+1][k-1] + dp[k][j]` 隐式实现 (对比"不合并" 应该是 `dp[i+1][k-1] + dp[k][j] + 0`... 想清楚就是"不加 1" 即节省一次).

    等价推导:
    - **不合并**: 打 t[i] (1 次) + 中间 (`dp[i+1][k-1]`) + t[k..j] (`dp[k][j]`).
    - **合并 t[i] 与 t[k]**: 中间 + t[k..j] — 打 t[i] 的那 1 次跟 dp[k][j] 里 t[k] 的第一次共享.
    - → 少 1 次 → `dp[i+1][k-1] + dp[k][j]` 正好.

    > **区间 DP 的"合并" 洞察是 Hard 常见考点**. 类似 [0312 Burst Balloons](../0312-burst-balloons/README.md) 的"最后戳" 反向枚举.

5. **🔑 遍历顺序: `len` 从 2 到 n, i 顺序, j = i+len-1 / Interval DP loop order**

    ```cpp
    for (int len = 2; len <= n; ++len)
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            ... 计算 dp[i][j] ...
        }
    ```

    保证**计算 `dp[i][j]`** 时**所有更小区间**已算好.

    > **区间 DP 的标志**: `len` 外层, i 内层. 少数题外层 i 倒序 + 内层 j 正序也行 (跟 [0516 LPS](../0516-longest-palindromic-subsequence/README.md) 一样).

6. **🔑 k 枚举分割点 / k enumerates match position**

    ```cpp
    for (int k = i + 1; k <= j; k++) {
        if (t[k] == t[i]) {
            dp[i][j] = min(dp[i][j], dp[i + 1][k - 1] + dp[k][j]);
        }
    }
    ```

    - 只**匹配上 t[i]** 的 k 才能合并 — 别的 k 跳过.
    - 试所有可能匹配位, 取最小.

7. **🔑 复杂度 O(n³) 时间, O(n²) 空间 / Cubic time, quadratic space**

    - 区间数: O(n²).
    - 每区间试 O(n) 个 k.
    - Total: O(n³). LC 允许.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int strangePrinter(string s) {
            // 预处理: 去连续重复
            string t;
            for (char c : s) if (t.empty() || c != t.back()) t.push_back(c);
            int n = t.size();

            vector<vector<int>> dp(n, vector<int>(n, 0));
            for (int i = 0; i < n; i++) dp[i][i] = 1;                // base: 单字符

            for (int len = 2; len <= n; ++len) {
                for (int i = 0; i + len - 1 < n; i++) {
                    int j = i + len - 1;
                    dp[i][j] = dp[i + 1][j] + 1;                     // 默认: 单独打 t[i]
                    for (int k = i + 1; k <= j; k++) {
                        if (t[k] == t[i]) {                          // 合并同色
                            dp[i][j] = min(dp[i][j], dp[i + 1][k - 1] + dp[k][j]);
                        }
                    }
                }
            }
            return dp[0][n - 1];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def strangePrinter(self, s: str) -> int:
            # 去连续重复
            t = []
            for c in s:
                if not t or c != t[-1]:
                    t.append(c)
            n = len(t)

            dp = [[0] * n for _ in range(n)]
            for i in range(n):
                dp[i][i] = 1
            for length in range(2, n + 1):
                for i in range(n - length + 1):
                    j = i + length - 1
                    dp[i][j] = dp[i + 1][j] + 1
                    for k in range(i + 1, j + 1):
                        if t[k] == t[i]:
                            # k == i+1 时 dp[i+1][k-1] = dp[i+1][i] — 未写入的格, 初始为 0.
                            # 语义: 中间段空, 打印数 = 0. 跟 C++ 一致
                            dp[i][j] = min(dp[i][j], dp[i + 1][k - 1] + dp[k][j])
            return dp[0][n - 1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var strangePrinter = function(s) {
        const t = [];
        for (const c of s) {
            if (t.length === 0 || c !== t[t.length - 1]) t.push(c);
        }
        const n = t.length;
        // Array.from + 工厂避免共享引用
        const dp = Array.from({length: n}, () => new Array(n).fill(0));
        for (let i = 0; i < n; i++) dp[i][i] = 1;
        for (let len = 2; len <= n; len++) {
            for (let i = 0; i + len - 1 < n; i++) {
                const j = i + len - 1;
                dp[i][j] = dp[i + 1][j] + 1;
                for (let k = i + 1; k <= j; k++) {
                    if (t[k] === t[i]) {
                        // k == i+1 时 dp[i+1][k-1] = dp[i+1][i] — 未写但初始 0, 同 C++
                        dp[i][j] = Math.min(dp[i][j], dp[i + 1][k - 1] + dp[k][j]);
                    }
                }
            }
        }
        return dp[0][n - 1];
    };
    ```

## Complexity

- **Time**: O(n³) — 区间数 × 每区间试 n 个 k.
- **Space**: O(n²) — dp 表.

## 相关题目

- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP + "最后戳" 枚举母题
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 区间 DP
- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 区间 DP 判定
- [1000. Minimum Cost to Merge Stones](../1000-minimum-cost-to-merge-stones/README.md) — 3D 区间 DP
- [1547. Minimum Cost to Cut a Stick](../1547-minimum-cost-to-cut-a-stick/README.md) — 区间 + 排序
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — 区间 min-max 博弈
- 0546\. Remove Boxes (待补, Hard×2) — 区间 DP + 三维状态 (超难变体)
- 1246\. Palindrome Removal (待补) — 区间 DP + 回文合并
- 0087\. Scramble String (待补) — 区间 DP + memo

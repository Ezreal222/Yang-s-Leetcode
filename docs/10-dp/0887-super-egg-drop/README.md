# 0887. Super Egg Drop / 鸡蛋掉落

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: DP, Dimension Flip, Binary Search, Classic · 动态规划, 反向思维, 二分, 经典
    - **Link**: [LeetCode](https://leetcode.com/problems/super-egg-drop/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min moves to find critical floor with k eggs, n floors** → 4 evolutions: **v1 classic O(k·n²)** try each first-drop floor; **v2 + binary search O(k·n log n)** exploit monotonicity; **v3 dimension-flip O(k·n)** — redefine as `dp[i][m] = max floors solvable with i eggs, m moves`; **v4 rolling to O(k) space**.
>
> **中文**: **k 蛋 n 楼, 最少几次找到临界楼** → 四级进化: **v1 O(k·n²)** 枚举首扔楼; **v2 + 二分 O(k·n log n)** 用单调性; **v3 反向定义 O(k·n)** — `dp[i][m]` = **i 蛋 m 步能测多少楼**; **v4 一维空间 O(k)**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: `k` 个鸡蛋, `n` 层楼, 存在临界楼层 `f` (从 f+1 开始扔会碎, ≤ f 不碎). 每次扔一颗蛋 (从任意楼), **观察是碎是不碎**, 蛋碎了不能再用. 求**最坏情况下最少几次投掷**能保证找出 f.

**中文**: k 蛋 n 楼求"最少投掷次数最坏保证找出临界".

## Key Insights

1. **🔑 v1 灵魂定义: `dp[i][j]` = i 蛋 j 楼最少次数 / dp[eggs][floors] = min drops**

    **转移**: 枚举**首次扔第 x 层** (1 ≤ x ≤ j) 的最坏结果:

    - 碎了: 蛋 -1, 楼 x-1 (下面 1..x-1) → `dp[i-1][x-1]`.
    - 没碎: 蛋不变, 楼 j-x (上面 x+1..j, 相对第 x+1 层重新编号) → `dp[i][j-x]`.
    - 最坏: `max(...)` + 1 (本次投掷).
    - 最优 x: 让最坏最小 → `dp[i][j] = min over x of (1 + max(dp[i-1][x-1], dp[i][j-x]))`.

    **base**: `dp[1][j] = j` (1 蛋只能**从 1 楼线性试**, 最坏扔 j 次).

    → **O(k · n²)** 时间, TLE for large n.

    > **典型 min-max DP**. 转移里的 max 是"最坏情况", 外层 min 是"最优策略".

2. **🔑 v2 二分优化: `dp[i-1][x-1]` 递增, `dp[i][j-x]` 递减 → 找交点 / Binary search on monotonic**

    固定 (i, j), 让 x 从 1 到 j:

    - `dp[i-1][x-1]` 随 x **递增** (楼多要更多次).
    - `dp[i][j-x]` 随 x **递减** (剩余楼少).
    - `max(...)` 是**先减后增的谷型** → 谷底 = 最优.

    → **二分找**最小 x 使 `dp[i-1][x-1] >= dp[i][j-x]`. 那个 x 或 x-1 就是谷底.

    → **O(k · n · log n)**, LC 能过.

3. **🔑 v3 灵魂反转: 定义反转维度, 反变正 / Dimension flip: reverse the definition**

    **换个问法**: **"i 蛋 m 步最多能测多少楼?"** → `dp[i][m] = ?`

    **转移**: 一次投掷:

    - 碎了: `dp[i-1][m-1]` 楼 (下面能测多少).
    - 没碎: `dp[i][m-1]` 楼 (上面能测多少).
    - 加当前投掷的**这一层** = 1.

    → **`dp[i][m] = dp[i-1][m-1] + dp[i][m-1] + 1`**.

    **答案**: 最小的 m 使 `dp[k][m] >= n`.

    ```
    n = 6, k = 2
    m=1: dp[1][1]=1, dp[2][1]=1
    m=2: dp[1][2]=2, dp[2][2]=1+1+1=3   ← 2 蛋 2 步能测 3 楼 (<6)
    m=3: dp[1][3]=3, dp[2][3]=2+3+1=6   ← 2 蛋 3 步能测 6 楼 (≥6) → 返 3
    ```

    → **O(k · n)** 时间. 从三层循环降到两层, **飞跃**.

    > **"求最小 X 使得 Y" 类** 常常可以反转成"给 X 求最大 Y" — **dimension flip**. Zuma / Egg Drop / 部分博弈 DP 都有这招. 是**DP 高阶技巧**.

4. **🔑 v4 一维空间: 倒序更新 / v4 rolling array**

    v3 的 `dp[i][m]` 只依赖 `dp[i-1][m-1]` 和 `dp[i][m-1]`. 用**一维 `f[i]`**代表当前 m 下的 dp[i][m]:

    ```cpp
    for (int i = k; i >= 1; --i)
        f[i] = f[i] + f[i - 1] + 1;
    //   ^^^^   ^^^^   ^^^^^^^^
    //   新 dp[i][m]   旧 dp[i][m-1]  旧 dp[i-1][m-1] (i 倒序保证是"旧" 值)
    ```

    - **倒序 i**: 保证 `f[i-1]` 仍是**上一轮的 dp[i-1][m-1]**, 未被更新.
    - **正序 i** 会用到已更新的 f[i-1] (新), 语义错.

    → **O(k)** 额外空间. 跟**0/1 背包一维版**同款倒序技巧.

    > **降维空间优化 = "看依赖方向决定顺序"** — 记熟 0/1 背包的倒序模板, 这题秒懂.

5. **🔑 四版对比 / Four-version summary**

    | 版本 | Time | Space | 思路核心 |
    |---|---|---|---|
    | v1 经典 | O(k · n²) | O(k · n) | 枚举首扔楼 |
    | v2 + 二分 | O(k · n log n) | O(k · n) | 单调性谷底 |
    | **v3 反向定义** | **O(k · n)** | O(k · n) | **dimension flip** |
    | **v4 一维滚动** | **O(k · n)** | **O(k)** | 倒序空间优化 |

    > **v3 → v4 是 LC 面试标准解**. v1 → v2 → v3 是**思维进化史**, 全懂了这题就精通了.

6. **🔑 base cases / Base cases**

    - v1: `dp[1][j] = j` (1 蛋只能线性), `dp[i][0] = 0` (0 楼 0 步), `dp[i][1] = 1`.
    - v3/v4: `dp[i][0] = 0` (0 步测 0 楼), `dp[0][m] = 0` (0 蛋测 0 楼).

    > **base 想清楚了 DP 就写出一半**.

7. **🔑 复杂度对比 (Hard 题标准) / Complexity comparison**

    - k = 100, n = 10000 时:
        - v1: 10^10 → TLE.
        - v2: 10^8 → 险过.
        - **v3/v4: 10^6 → 稳过**.

## Solution

=== "C++"

    **v1: 经典 O(k · n²)**

    ```cpp
    class Solution {
    public:
        int superEggDrop(int k, int n) {
            vector<vector<int>> dp(k + 1, vector<int>(n + 1, 0));
            for (int j = 1; j <= n; j++) dp[1][j] = j;
            for (int i = 2; i <= k; i++) {
                for (int j = 1; j <= n; j++) {
                    dp[i][j] = INT_MAX;
                    for (int x = 1; x <= j; x++) {
                        dp[i][j] = min(dp[i][j], 1 + max(dp[i - 1][x - 1], dp[i][j - x]));
                    }
                }
            }
            return dp[k][n];
        }
    };
    ```

    **v2: + 二分 O(k · n log n)**

    ```cpp
    class Solution {
    public:
        int superEggDrop(int k, int n) {
            vector<vector<int>> dp(k + 1, vector<int>(n + 1, 0));
            for (int j = 1; j <= n; j++) dp[1][j] = j;
            for (int i = 2; i <= k; i++) {
                for (int j = 1; j <= n; j++) {
                    int lo = 1, hi = j;
                    while (lo < hi) {
                        int mid = lo + (hi - lo) / 2;
                        if (dp[i - 1][mid - 1] < dp[i][j - mid]) lo = mid + 1;
                        else hi = mid;
                    }
                    dp[i][j] = 1 + max(dp[i - 1][lo - 1], dp[i][j - lo]);
                }
            }
            return dp[k][n];
        }
    };
    ```

    **v3: 反向定义 O(k · n)**

    ```cpp
    class Solution {
    public:
        int superEggDrop(int k, int n) {
            vector<vector<int>> dp(k + 1, vector<int>(n + 1, 0));
            // dp[i][m] = i 蛋 m 步能测最多多少楼
            for (int m = 1; m <= n; m++) {
                for (int i = 1; i <= k; i++) {
                    dp[i][m] = dp[i - 1][m - 1] + dp[i][m - 1] + 1;
                }
                if (dp[k][m] >= n) return m;
            }
            return n;
        }
    };
    ```

    **v4: 一维滚动 O(k) 空间**

    ```cpp
    class Solution {
    public:
        int superEggDrop(int k, int n) {
            vector<int> f(k + 1, 0);
            int m = 0;
            while (f[k] < n) {
                ++m;
                for (int i = k; i >= 1; --i)                         // 倒序 i
                    f[i] = f[i] + f[i - 1] + 1;
            }
            return m;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v4 — 最简一维版
        def superEggDrop(self, k: int, n: int) -> int:
            f = [0] * (k + 1)
            m = 0
            while f[k] < n:
                m += 1
                # 倒序 i, 保证 f[i-1] 仍是上一轮值
                for i in range(k, 0, -1):
                    f[i] = f[i] + f[i - 1] + 1
            return m
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} k
     * @param {number} n
     * @return {number}
     */
    var superEggDrop = function(k, n) {
        // v4
        const f = new Array(k + 1).fill(0);
        let m = 0;
        while (f[k] < n) {
            m++;
            for (let i = k; i >= 1; i--) {
                f[i] = f[i] + f[i - 1] + 1;
            }
        }
        return m;
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| v1 | O(k · n²) | O(k · n) |
| v2 (二分) | O(k · n log n) | O(k · n) |
| v3 (反向) | O(k · n) | O(k · n) |
| **v4 (一维)** | **O(k · n)** | **O(k)** |

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP 母题
- [0072. Edit Distance](../0072-edit-distance/README.md) — 二维 DP 经典
- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包
- [0300. Longest Increasing Subsequence](../0300-longest-increasing-subsequence/README.md) — LIS, 含 O(n log n) 二分
- [0312. Burst Balloons](../0312-burst-balloons/README.md) — 区间 DP + 枚举最后
- [0375. Guess Number Higher or Lower II](../0375-guess-number-higher-or-lower-ii/README.md) — min-max 博弈 DP
- 1884\. Egg Drop With 2 Eggs and N Floors (待补) — k=2 特例
- 0664\. Strange Printer (待补) — 区间 DP + 反向
- 0552\. Student Attendance Record II (待补) — 一维 DP 优化

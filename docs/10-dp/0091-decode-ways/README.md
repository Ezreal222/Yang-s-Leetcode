# 0091. Decode Ways / 解码方法

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Linear, String · 动态规划, 线性, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/decode-ways/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Digit string → letter (A=1..Z=26), count decodings** → **linear DP** `dp[i]` = ways for `s[0..i-1]`; two transitions: **1-char** (`s[i-1] ≠ '0'` ⇒ `+= dp[i-1]`), **2-char** (`10 ≤ s[i-2..i-1] ≤ 26` ⇒ `+= dp[i-2]`). Handles `'0'` implicitly: if neither transition fires ⇒ 0 ways.
>
> **中文**: **数字串编码字母求解码方案数** → **线性 DP** `dp[i]` = 前 i 字符方案数; 两种转移: **单字符** (非 0 → `+= dp[i-1]`), **双字符** (10-26 → `+= dp[i-2]`). 0 的处理隐含: 都不满足则 dp[i] = 0.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数字串 `s`. 编码规则 `A=1, B=2, ..., Z=26`. 求 `s` 的**解码方案数**.

- 例: `"12"` → 2 (`"AB"` = 1+2, `"L"` = 12).
- 例: `"27"` → 1 (只能 `"BG"`, 因为 27 > 26).
- 例: `"06"` → 0 (0 开头无解).

**中文**: 数字串 → 字母的解码方案数.

## Key Insights

1. **🔑 灵魂: 每步 1 字符 或 2 字符 (两种子结构) / Every step: 1-char OR 2-char (two subproblems)**

    到位置 i, 最后一段解码可以是:

    - **单字符** (`s[i-1]`): 前 i-1 已解码, 加上单字符 `s[i-1]`. 前提 `s[i-1] ≠ '0'` (0 单独无解).
    - **双字符** (`s[i-2..i-1]`): 前 i-2 已解码, 加上双字符. 前提两字符值 ∈ [10, 26].

    → **`dp[i] = (可单 ? dp[i-1] : 0) + (可双 ? dp[i-2] : 0)`**.

    > **线性 DP 双转移** — 跟 [0198 House Robber](../0198-house-robber/README.md) 的"选 / 不选" 结构类似, 只是这里两种都可能成立 → 求和.

2. **🔑 base cases: `dp[0] = 1, dp[1] = 1` / Base**

    - **`dp[0] = 1`**: 空串**有 1 种"空解码"** (哨兵, 让双字符转移有意义).
    - **`dp[1] = 1`**: 前 1 字符**若非 0** 就 1 种 (Yang 用 `s[0] == '0'` 早退保证).

    > **`dp[0] = 1` 是哨兵语义**: 让 dp[2] = dp[0] 时表示"整个 s[0..1] 作为双字符".

3. **🔑 0 的处理: 隐含在条件里 / Zero handled implicitly**

    - **`s[i-1] == '0'`** → 单字符不成立, 只能靠双字符.
    - **`s[i-2..i-1] ∈ [10, 26]`** — 若 `s[i-2] == '0'`, 组合是 `0X`, 不在 [10, 26] → 双字符也不成立.
    - **两者都不满足** → `dp[i] = 0` → 后续全部继承为 0 → 最终返 0.

    Yang 的代码利用**默认 dp[i] = 0** 覆盖这条逻辑, **无需显式判**"该位置 0 又没前导" 的死锁.

    > **"用 dp 值传播 0" 是精简写法**. 若手动判"死锁", 分支多且累赘.

4. **🔑 Yang 的写法逐行 / Line-by-line**

    ```cpp
    if (s[0] == '0') return 0;                   // 早退, 不用维护 dp[1] = 0 的边界
    vector<int> dp(n + 1, 0);
    dp[0] = 1; dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        if (s[i - 1] != '0') dp[i] = dp[i - 1];             // 单字符
        int two = (s[i - 2] - '0') * 10 + (s[i - 1] - '0');
        if (two >= 10 && two <= 26) dp[i] += dp[i - 2];     // 双字符
    }
    return dp[n];
    ```

    - **`dp[i] = dp[i-1]`** (赋值) — 单字符分支.
    - **`dp[i] += dp[i-2]`** (累加) — 双字符分支 (可能跟单字符共存).

5. **🔑 边界 case 走一遍 / Corner cases traced**

    | 输入 | 单/双 是否成立 | dp | 答案 |
    |---|---|---|---|
    | `"06"` | s[0]='0' 早退 | — | **0** |
    | `"10"` | i=2: 单 (s[1]='0'❌), 双 (10 ✓ → dp[0]=1) | [1,1,1] | **1** |
    | `"12"` | i=2: 单 (s[1]='2'✓ → dp[1]=1), 双 (12 ✓ → dp[0]=1) | [1,1,2] | **2** |
    | `"27"` | i=2: 单 (s[1]='7'✓ → dp[1]=1), 双 (27❌) | [1,1,1] | **1** |
    | `"100"` | i=2: dp[2]=1 (只双). i=3: 单 (s[2]='0'❌), 双 (00❌) → dp[3]=0 | [1,1,1,0] | **0** |

    > **手 trace 几个 corner case** 是理解 DP 的关键练习.

6. **🔑 空间优化 O(1) / Space optimization**

    `dp[i]` 只依赖 `dp[i-1]` 和 `dp[i-2]` → 用两个变量滚动:

    ```cpp
    int prev2 = 1, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int cur = 0;
        if (s[i-1] != '0') cur = prev1;
        int two = (s[i-2] - '0') * 10 + (s[i-1] - '0');
        if (two >= 10 && two <= 26) cur += prev2;
        prev2 = prev1; prev1 = cur;
    }
    return prev1;
    ```

    > **一维 DP 只依赖前 k 项 → 滚动 O(k) 空间** — 跟 0198 / 0509 同源.

7. **🔑 复杂度 O(n) 时间, O(n) 空间 / Linear**

    - Time: 一遍扫.
    - Space: dp 数组 (可优化 O(1)).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int numDecodings(string s) {
            int n = s.size();
            if (s[0] == '0') return 0;                              // 早退

            vector<int> dp(n + 1, 0);
            dp[0] = 1; dp[1] = 1;

            for (int i = 2; i <= n; i++) {
                if (s[i - 1] != '0') dp[i] = dp[i - 1];             // 单字符
                int two = (s[i - 2] - '0') * 10 + (s[i - 1] - '0');
                if (two >= 10 && two <= 26) dp[i] += dp[i - 2];     // 双字符 (可能累加)
            }
            return dp[n];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def numDecodings(self, s: str) -> int:
            n = len(s)
            if s[0] == '0': return 0
            dp = [0] * (n + 1)
            dp[0] = dp[1] = 1
            for i in range(2, n + 1):
                if s[i - 1] != '0':
                    dp[i] = dp[i - 1]
                two = int(s[i - 2:i])       # 切片 + int 一步拿两位数 — 比 (s[i-2]-'0')*10 + ... 更简洁
                if 10 <= two <= 26:
                    dp[i] += dp[i - 2]
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var numDecodings = function(s) {
        const n = s.length;
        if (s[0] === '0') return 0;
        const dp = new Array(n + 1).fill(0);
        dp[0] = 1; dp[1] = 1;
        for (let i = 2; i <= n; i++) {
            if (s[i - 1] !== '0') dp[i] = dp[i - 1];
            // parseInt(s.substring(i-2, i)) 一步取两位数
            const two = parseInt(s.substring(i - 2, i), 10);
            if (two >= 10 && two <= 26) dp[i] += dp[i - 2];
        }
        return dp[n];
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫.
- **Space**: O(n) — dp 数组, 可优化 O(1).

## 相关题目

- [0198. House Robber](../0198-house-robber/README.md) — 线性 DP 母题 (选 / 不选)
- [0070. Climbing Stairs](../0070-climbing-stairs/README.md) — 一维 DP 基础 (1/2 步)
- [0746. Min Cost Climbing Stairs](../0746-min-cost-climbing-stairs/README.md) — 花费型线性 DP
- [0983. Minimum Cost For Tickets](../0983-minimum-cost-for-tickets/README.md) — 三种转移
- [0509. Fibonacci Number](../0509-fibonacci-number/README.md) — 一维 DP 最简
- [0139. Word Break](../0139-word-break/README.md) — 线性 DP 判定 (字符串)
- 0639\. Decode Ways II (待补) — 含 `*` 通配符, DP 更复杂
- 0940\. Distinct Subsequences II (待补) — DP + 去重

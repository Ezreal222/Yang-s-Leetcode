# 0005. Longest Palindromic Substring / 最长回文子串

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval DP, Center Expansion, String · 动态规划, 区间 DP, 中心扩展, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-palindromic-substring/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Longest palindromic substring** → **v1 center expansion** O(n²) time, **O(1) space**: try each position as center (odd) and each pair as center (even), expand outward. **v2 interval DP** O(n²) time/space: `dp[i][j] = (s[i]==s[j]) && (j-i<2 || dp[i+1][j-1])`.
>
> **中文**: **最长回文子串** → **v1 中心扩展** O(n²) 时间 **O(1) 空间**: 每位置作奇/偶中心两端扩. **v2 区间 DP** O(n²): `dp[i][j] = 两端相等 && 内部回文`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 字符串 `s`, 找**最长的回文子串**.

**中文**: 找 s 的最长回文子串.

## Key Insights

1. **🔑 v1 灵魂: 每位置作"中心" 向外扩 / Center expansion**

    回文有**中心对称性**: 长为 L 的回文有**唯一中心**:

    - **奇长 (L 奇)**: 中心是**单字符** (n 个位置).
    - **偶长 (L 偶)**: 中心是**两字符间**的缝 (n-1 个位置).

    → 遍历 **2n-1 个中心**, 每个向两端扩到不匹配, 记最长.

    ```cpp
    void expand(l, r):
        while (l >= 0 && r < n && s[l] == s[r]) { l--; r++; }
        // 停下时 [l+1, r-1] 是回文, 长 = r - l - 1
    ```

    - **奇中心**: `expand(i, i)` (两指针都从 i 开始, 单字符起步).
    - **偶中心**: `expand(i, i+1)` (i 和 i+1 起步).

    > **中心扩展 = O(n²) 时间 O(1) 空间的经典** — 比 DP 更优雅.

2. **🔑 v1 记录: `start` 和 `maxLen` / Track best**

    停下时 `[l+1, r-1]` 是回文, **长 `r - l - 1`**. 若大于 maxLen, 更新 `start = l + 1, maxLen = r - l - 1`.

    最终 `s.substr(start, maxLen)` 返回子串.

3. **🔑 v2 区间 DP: `dp[i][j]` = s[i..j] 是回文? / Interval DP**

    **转移**:

    ```cpp
    dp[i][j] = (s[i] == s[j]) &&
               (j - i < 2 || dp[i + 1][j - 1])
    ```

    - **两端相等** 是必要条件.
    - **内部是回文** — 若长 ≤ 2 (`j - i < 2`), 内部为空或单字符, 自动回文; 否则查 `dp[i+1][j-1]`.

    **遍历**: **len 从小到大** (2 到 n), 保证 dp[i+1][j-1] (更短) 先算好.

    > **区间 DP 标志顺序**: len 外层, i 内层. 跟 [0647 Palindromic Substrings](../0647-palindromic-substrings/README.md) / [0516 LPS](../0516-longest-palindromic-subsequence/README.md) 同族.

4. **🔑 v2 base cases / Base cases**

    - **`dp[i][i] = true`** — 单字符必回文.
    - **`dp[i][i+1] = (s[i] == s[i+1])`** — 两字符靠字符相等.
    - 上面转移的 `j - i < 2` 分支覆盖了这两个 base.

5. **🔑 v1 vs v2 对比 / v1 vs v2**

    | | **v1 中心扩展** | **v2 区间 DP** |
    |---|---|---|
    | Time | O(n²) | O(n²) |
    | **Space** | **O(1)** | O(n²) |
    | 代码 | 短 | 中 |
    | 面试推荐 | **首选** | 展示 DP 思维 |
    | 教学价值 | 中 | **高** (标准区间 DP) |

    > **面试首选 v1** (空间更优). **教学价值 v2 更高** (练区间 DP 顺序 + 状态设计).

6. **🔑 进阶 O(n) Manacher / Manacher O(n)**

    **Manacher 算法** 可在 **O(n)** 内解决. 核心思想: 利用**已知回文的对称性** 复用信息, 避免重复扩展. 代码 30 行, 复杂. **LC 不需要**, 学有余力再看.

    > "Manacher" 是 LC 里少数**面试不必掌握** 的进阶算法之一.

7. **🔑 跟其他回文题的关系 / Palindrome family**

    | 题 | 找什么 | 方法 |
    |---|---|---|
    | [0647 Palindromic Substrings](../0647-palindromic-substrings/README.md) | 数 (计数) | 中心扩展或 DP |
    | **0005 (本题)** | **最长** 子串 | 中心扩展或 DP |
    | [0516 LPS](../0516-longest-palindromic-subsequence/README.md) | 最长子**序列** | 区间 DP |
    | [1312 Min Insertions](../1312-minimum-insertion-steps-to-make-a-string-palindrome/README.md) | 变回文最小插入 | LPS 镜像 |
    | [0125 Valid Palindrome](../../05-two-pointers/0125-valid-palindrome/README.md) | 判 | 对撞双指针 |
    | [0266 Palindrome Permutation](../../03-hash-table/0266-palindrome-permutation/README.md) | 判可回文 | 频次 |

    > **回文一族**围绕"中心对称" 展开, 但**方法各不同**.

8. **🔑 复杂度 / Complexity**

    - **Time**: O(n²) 两版都.
    - **Space**: **O(1) v1** / O(n²) v2.

## Solution

=== "C++"

    **v1: 中心扩展 (推荐, O(1) 空间)**

    ```cpp
    class Solution {
    public:
        int start = 0, maxLen = 0;
        void expand(const string& s, int l, int r) {
            while (l >= 0 && r < (int)s.size() && s[l] == s[r]) { l--; r++; }
            if (r - l - 1 > maxLen) {                                // 停下时 [l+1, r-1] 是回文
                start = l + 1;
                maxLen = r - l - 1;
            }
        }
        string longestPalindrome(string s) {
            for (int i = 0; i < (int)s.size(); i++) {
                expand(s, i, i);                                     // 奇中心
                expand(s, i, i + 1);                                 // 偶中心
            }
            return s.substr(start, maxLen);
        }
    };
    ```

    **v2: 区间 DP (教学清晰)**

    ```cpp
    class Solution {
    public:
        string longestPalindrome(string s) {
            int n = s.size(), start = 0, maxLen = 1;
            vector<vector<bool>> dp(n, vector<bool>(n, false));
            for (int i = 0; i < n; i++) dp[i][i] = true;             // 单字符

            for (int len = 2; len <= n; len++) {
                for (int i = 0; i + len - 1 < n; i++) {
                    int j = i + len - 1;
                    if (s[i] != s[j]) continue;                      // 两端不等 → 非回文
                    if (len == 2 || dp[i + 1][j - 1]) {              // 内部空或已确认回文
                        dp[i][j] = true;
                        if (len > maxLen) {
                            start = i;
                            maxLen = len;
                        }
                    }
                }
            }
            return s.substr(start, maxLen);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1 中心扩展 — 最简洁
        def longestPalindrome(self, s: str) -> str:
            self.start = 0
            self.max_len = 0
            n = len(s)

            def expand(l: int, r: int) -> None:
                while l >= 0 and r < n and s[l] == s[r]:
                    l -= 1
                    r += 1
                if r - l - 1 > self.max_len:
                    self.start = l + 1
                    self.max_len = r - l - 1

            for i in range(n):
                expand(i, i)          # 奇
                expand(i, i + 1)      # 偶
            return s[self.start:self.start + self.max_len]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string}
     */
    var longestPalindrome = function(s) {
        // v1 中心扩展 — 闭包捕获 start / maxLen 更清晰
        let start = 0, maxLen = 0;
        const n = s.length;
        const expand = (l, r) => {
            while (l >= 0 && r < n && s[l] === s[r]) {
                l--; r++;
            }
            if (r - l - 1 > maxLen) {
                start = l + 1;
                maxLen = r - l - 1;
            }
        };
        for (let i = 0; i < n; i++) {
            expand(i, i);
            expand(i, i + 1);
        }
        return s.substring(start, start + maxLen);
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| **v1 中心扩展** | O(n²) | **O(1)** |
| v2 区间 DP | O(n²) | O(n²) |
| Manacher (进阶) | **O(n)** | O(n) |

## 相关题目

- [0647. Palindromic Substrings](../0647-palindromic-substrings/README.md) — 数回文子串数 (同族)
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 最长回文**子序列** (LPS)
- [1312. Minimum Insertion Steps to Make a String Palindrome](../1312-minimum-insertion-steps-to-make-a-string-palindrome/README.md) — LPS 镜像
- [0664. Strange Printer](../0664-strange-printer/README.md) — 区间 DP + 合并同色
- [0125. Valid Palindrome](../../05-two-pointers/0125-valid-palindrome/README.md) — 判回文
- [0266. Palindrome Permutation](../../03-hash-table/0266-palindrome-permutation/README.md) — 判可回文排列
- [0267. Palindrome Permutation II](../../08-backtracking/0267-palindrome-permutation-ii/README.md) — 生成所有回文排列
- 0409\. Longest Palindrome (待补) — 频次构造
- 0131\. Palindrome Partitioning (待补) — 划分成回文子串
- 0132\. Palindrome Partitioning II (待补) — 最少划分
- 0234\. Palindrome Linked List (待补) — 链表版

# 0647. Palindromic Substrings / 回文子串

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Interval, Two Pointers, String · 动态规划, 区间, 双指针, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/palindromic-substrings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s`, 返回**回文子串的总数** (不同位置或不同长度算不同子串).

**中文**: 统计 `s` 中**回文子串个数** (按位置区分).

## Key Insights

1. **🔑 第一道区间 DP / First interval DP**

    跟之前的"前 i / 以 i 结尾" 状态都不一样, 本题 `dp[i][j]` 关心的是**一段区间 `[i, j]`**. 这是区间 DP 的入门:

    > **看到"子串 / 子区间 / 段" + 判定/计数** → 想 `dp[i][j]` 区间状态.

2. **状态: `dp[i][j] = s[i..j] 是否回文` (bool) / Interval boolean state**

    比较 [0718 子数组](../0718-maximum-length-of-repeated-subarray/README.md) 的"以 i, j 同时结尾", 这里两个下标含义换了 — `i, j` 是**区间端点**, 不是结尾.

3. **🔑 转移: 末位相等 + 内部回文 / Recurrence**

    `s[i..j]` 是回文需要两个条件:

    - **两端相等**: `s[i] == s[j]`
    - **内部也回文**: `s[i+1..j-1]` 是回文 (即 `dp[i+1][j-1]`)

    边界:

    - `j - i == 0` (单字符): 永远是回文.
    - `j - i == 1` (两字符): 两个一样就是回文, 不需要内部.
    - `j - i >= 2`: 需要 `dp[i+1][j-1]`.

    合起来一行:

    ```cpp
    dp[i][j] = (s[i] == s[j]) && (j - i <= 1 || dp[i + 1][j - 1]);
    ```

4. **🔑 遍历顺序: i 从大到小, j 从 i 到 n-1 / Reverse i, forward j**

    `dp[i][j]` 依赖 `dp[i+1][j-1]` — **i 更大的行先算好**, **j 更小的列先算好**. 所以:

    - **外层 `i` 从 `n-1` 到 `0`** (倒序, 先算更大 i 的状态).
    - **内层 `j` 从 `i` 到 `n-1`** (顺序, j 必 ≥ i).

    > **遍历方向必须保证依赖先算好**. 区间 DP 的方向跟之前"前 i / 以 i 结尾" 不一样, 容易写错. 验证方法: 画方格图, 标出 `[i+1][j-1]` 在 `[i][j]` 的哪边 → 必须先填那边.

    > 另一种等价写法是**按区间长度 `len = j - i + 1` 由小到大**: 外层 len, 内层 i, j = i + len - 1. 思维更"区间 DP". 留作进阶.

5. **边遍历边计数 / Count inline**

    每标到一个 `dp[i][j] = true`, `count++`. 不需要最后扫整张表 — 也可以扫, 一样的复杂度.

6. **替代解法: 中心扩散 O(n²) 不用 dp / Center expansion alternative**

    每个字符 / 每个相邻字符间隙都可能是回文中心. 对每个中心向两侧扩散, 数能匹配多远. 共 `2n - 1` 个中心 (n 个字符 + n-1 个间隙), 总 O(n²) 时间, **O(1) 空间** — 比 dp 省内存. 写法:

    ```cpp
    int expand(string& s, int l, int r) {
        int cnt = 0;
        while (l >= 0 && r < s.size() && s[l] == s[r]) { cnt++; l--; r++; }
        return cnt;
    }
    // 主循环: for (int i = 0; i < n; i++) count += expand(s, i, i) + expand(s, i, i + 1);
    ```

    > 同 O(n²) 复杂度但少 O(n²) 内存. 写法更短. 进阶可学 Manacher 算法 O(n), 但 LC 数据规模 DP / 扩散都够.

## Solution

=== "C++"
    === "v1 推荐: 区间 DP"
        ```cpp
        class Solution {
        public:
            int countSubstrings(string s) {
                int n = s.size(), count = 0;
                vector<vector<bool>> dp(n, vector<bool>(n, false));
                for (int i = n - 1; i >= 0; i--) {                 // ⚠ i 倒序
                    for (int j = i; j < n; j++) {                  // j 顺序, j ≥ i
                        if (s[i] == s[j] && (j - i <= 1 || dp[i + 1][j - 1])) {
                            dp[i][j] = true;
                            count++;
                        }
                    }
                }
                return count;
            }
        };
        ```

    === "v2: 中心扩散 O(1) 空间"
        ```cpp
        class Solution {
        public:
            int countSubstrings(string s) {
                int count = 0, n = s.size();
                auto expand = [&](int l, int r) {
                    while (l >= 0 && r < n && s[l] == s[r]) { count++; l--; r++; }
                };
                for (int i = 0; i < n; i++) {
                    expand(i, i);                                  // 奇数长度中心
                    expand(i, i + 1);                              // 偶数长度中心 (中间是间隙)
                }
                return count;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def countSubstrings(self, s: str) -> int:
            # v2 中心扩散版 — Pythonic + O(1) 空间
            count, n = 0, len(s)
            def expand(l, r):
                nonlocal count
                while l >= 0 and r < n and s[l] == s[r]:
                    count += 1
                    l -= 1
                    r += 1
            for i in range(n):
                expand(i, i)                                       # 奇数中心
                expand(i, i + 1)                                   # 偶数中心
            return count
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var countSubstrings = function(s) {
        let count = 0;
        const n = s.length;
        const expand = (l, r) => {
            while (l >= 0 && r < n && s[l] === s[r]) { count++; l--; r++; }
        };
        for (let i = 0; i < n; i++) {
            expand(i, i);
            expand(i, i + 1);
        }
        return count;
    };
    ```

## Complexity

- **Time**: O(n²).
- **Space**: O(n²) (v1) / O(1) (v2).

## 相关题目

- 0005\. Longest Palindromic Substring (待补) — 同 dp / 扩散, 求最长回文子串
- [0516. Longest Palindromic Subsequence](../0516-longest-palindromic-subsequence/README.md) — 子序列版, 同区间 DP 但允许跳
- 0131\. Palindrome Partitioning (待补) — 回文 + 回溯
- 0132\. Palindrome Partitioning II (待补) — 最少分割, 区间 DP
- 0125\. Valid Palindrome (待补) — 判定单串是否回文 (双指针)
- 0680\. Valid Palindrome II (待补) — 允许删一个字符的判定

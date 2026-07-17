# 0459. Repeated Substring Pattern / 重复的子字符串

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: KMP, String Matching, Border, Failure Function · KMP, 字符串匹配, 边界, 失配函数
    - **Link**: [LeetCode](https://leetcode.com/problems/repeated-substring-pattern/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Is `s` made of repeating a substring ≥ 2 times?** → **KMP `next[]` (border) trick**: let `L = next[n-1]` (longest proper prefix = suffix). `s` is periodic iff `L > 0 && n % (n - L) == 0` — where `n - L` is the period length.
>
> **中文**: **s 是不是由某子串重复 ≥ 2 次拼成?** → **KMP 失配函数 `next[]`**: 设 `L = next[n-1]` (最长真前缀 = 真后缀). 周期 ⇔ `L > 0 && n % (n - L) == 0`, 周期长度 = `n - L`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断 `s` 是否可以由**它的某个子串重复多次 (≥ 2 次)** 拼接得到.

- 例: "abab" ✅ ("ab" × 2), "aba" ❌, "abcabcabc" ✅ ("abc" × 3).

**中文**: 判 s 是不是某子串重复 ≥ 2 次拼出.

## Key Insights

1. **🔑 灵魂洞察: 周期串 ⇔ 最长 border 长度 L 满足 `(n - L) | n && L > 0` / Periodic ⇔ `(n - L) | n && L > 0`**

    数学:

    - 若 `s = pattern × k` (k ≥ 2), 则**最长真前缀 = 真后缀** 是 `pattern × (k-1)`, 长度 **`L = (k-1) × p`** where `p = |pattern|`.
    - 于是 **`p = n - L`**, 且 `n = k × p` → **`p | n`**.
    - 反之亦然 — 若 `n - L | n && L > 0`, s 一定周期 (KMP 定理).

    例: `s = "abcabcabc"`, n = 9. Border `L = "abcabc"` = 6. `p = 9 - 6 = 3` = `|"abc"|`. `9 % 3 = 0`. ✅

    > **KMP 的 next 数组不只用于匹配 — 还能秒判周期性**. 是 KMP 的"副作用" 应用.

2. **🔑 `L > 0` 守卫: 排除"无 border" 情况 / `L > 0` guard**

    - 若 `L == 0` → s 里**没有非平凡 border** → 不周期 (排除 "abcde" 这种).
    - 少写 `L > 0` → 会把 `n % n == 0` 误判为周期 → 单字符也返 true (错).

    > **`>0` 排除 trivial border**. 想清楚"最短的一段" 是不是等于自己就懂了.

3. **🔑 KMP `next` 数组定义 / KMP `next[i]` definition**

    ```
    next[i] = 使 s[0..k-1] == s[i-k+1..i] (真前缀 = 真后缀) 的最大 k, 且 k < i+1.
    ```

    - "真" 前缀 / 后缀: **不含自身**.
    - 若无匹配, next[i] = 0.
    - 例: "aabaa" → next = [0, 1, 0, 1, 2].

    > **next[i] 是 s[0..i] 这段的最长自我重复**. 匹配失败时"跳回 next[i]" 而不是从头, 是 KMP 的核心.

4. **🔑 构建 `next` 的双指针 pattern / Building next with two pointers**

    ```cpp
    int j = 0;                              // j = 当前"最长匹配前缀" 的末位下标
    for (int i = 1; i < n; i++) {
        while (j > 0 && s[i] != s[j])
            j = next[j - 1];                 // 失配 → 沿 next 链回退
        if (s[i] == s[j]) j++;               // 匹配 → 前缀延长
        next[i] = j;                         // 记录
    }
    ```

    - **`i`** 从 1 起 (next[0] = 0).
    - **`j`** 是当前候选前缀的**尾**.
    - **失配就回退 `j = next[j-1]`** — 沿 border 链找短一点的候选.
    - **匹配就 j++**.

    > **KMP 构建 next 的模板背下来**. 6 行代码, 10 分钟看透, 一辈子受用.

5. **🔑 备选招: `(s + s).substr(1, 2n - 2).find(s) != npos` / Shift trick**

    经典 hack:

    ```cpp
    string t = s + s;
    return t.substr(1, 2 * s.size() - 2).find(s) != string::npos;
    ```

    直觉: 若 s 周期, `s + s` 里错位一位后还能找到 s. 例:

    ```
    s = "abab"
    s + s = "abababab"
    去头去尾 "bababa"
    包含 s "abab"?  ✅ 是 (从下标 1 开始)
    ```

    - **简单直观**, 不用理解 KMP.
    - **代码 3 行**, `.find()` 内部可以是 KMP → **本质仍是 KMP**.
    - **缺点**: 额外 O(n) 空间构造 `s + s`.

    > **面试**: 若不熟 KMP, 这招是**保命答案**. 熟 KMP → 用 next 更优雅.

6. **🔑 复杂度 O(n) 时间, O(n) 空间 / Linear**

    - **Time**: 构 next O(n) + 判定 O(1). 均摊 O(n) — 尽管 `while` 内层, KMP 的经典摊销.
    - **Space**: O(n) 存 next.

7. **🔑 KMP 家族路线图 / KMP roadmap**

    - **0459 (本题)**: KMP 的"副作用" 应用 (判周期).
    - **0028 Find the Index of the First Occurrence in a String** (待补): KMP 母题, 主串匹配模式串.
    - **0214 Shortest Palindrome** (待补): KMP 找"最长回文前缀".
    - **1392 Longest Happy Prefix** (待补): 直接返 next[n-1].

    > 学过 KMP 一次, 会开一族的门.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool repeatedSubstringPattern(string s) {
            int n = s.size();
            vector<int> next = getNext(s);
            int L = next[n - 1];
            return L > 0 && n % (n - L) == 0;                       // 灵魂两条件
        }
    private:
        vector<int> getNext(const string& s) {
            int n = s.size();
            vector<int> next(n, 0);
            int j = 0;                                              // 当前候选前缀末位
            for (int i = 1; i < n; i++) {
                while (j > 0 && s[i] != s[j])
                    j = next[j - 1];                                // 失配, 沿 next 回退
                if (s[i] == s[j]) j++;                              // 匹配, 前缀延长
                next[i] = j;
            }
            return next;
        }
    };

    // 备选: shift trick (3 行, 不需 KMP)
    class SolutionShift {
    public:
        bool repeatedSubstringPattern(string s) {
            string t = s + s;
            return t.substr(1, 2 * s.size() - 2).find(s) != string::npos;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # KMP next 版
        def repeatedSubstringPattern(self, s: str) -> bool:
            n = len(s)
            nxt = [0] * n
            j = 0
            for i in range(1, n):
                while j > 0 and s[i] != s[j]:
                    j = nxt[j - 1]
                if s[i] == s[j]:
                    j += 1
                nxt[i] = j
            L = nxt[n - 1]
            return L > 0 and n % (n - L) == 0

        # 备选: shift trick — Python 里更短
        def repeatedSubstringPattern_shift(self, s: str) -> bool:
            # (s + s)[1:-1] 去头去尾, 再判包含 s
            # `in` 内部对 str 是 O(n·m) 朴素 (CPython) 或 O(n) (若用 Boyer-Moore, 版本相关)
            return s in (s + s)[1:-1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {boolean}
     */
    var repeatedSubstringPattern = function(s) {
        // KMP next 版
        const n = s.length;
        const nxt = new Array(n).fill(0);
        let j = 0;
        for (let i = 1; i < n; i++) {
            while (j > 0 && s[i] !== s[j]) j = nxt[j - 1];
            if (s[i] === s[j]) j++;
            nxt[i] = j;
        }
        const L = nxt[n - 1];
        return L > 0 && n % (n - L) === 0;

        // 备选一行: return (s + s).slice(1, -1).includes(s);
    };
    ```

## Complexity

- **Time**: O(n) — KMP 构 next 是均摊线性.
- **Space**: O(n) — next 数组.

## 相关题目

- [0271. Encode and Decode Strings](../0271-encode-and-decode-strings/README.md) — 字符串编码设计
- [0415. Add Strings](../0415-add-strings/README.md) — 字符串加法
- [0067. Add Binary](../0067-add-binary/README.md) — 二进制加法
- [0187. Repeated DNA Sequences](../../03-hash-table/0187-repeated-dna-sequences/README.md) — 定长子串重复, 滑窗 + 滚动哈希
- [0266. Palindrome Permutation](../../03-hash-table/0266-palindrome-permutation/README.md) — 字符频次判可回文
- 0028\. Find the Index of the First Occurrence in a String (待补) — KMP 母题, 匹配模式串
- 0214\. Shortest Palindrome (待补) — KMP 找最长回文前缀
- 1392\. Longest Happy Prefix (待补) — 直接返 next[n-1]
- 0796\. Rotate String (待补) — s+s 判旋转

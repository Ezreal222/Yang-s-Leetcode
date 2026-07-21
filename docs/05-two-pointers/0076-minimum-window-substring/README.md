# 0076. Minimum Window Substring / 最小覆盖子串

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Sliding Window, Two Pointers, Hash Map, String · 滑动窗口, 双指针, 哈希表, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-window-substring/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Shortest substring of `s` covering all chars of `t` (with multiplicity)** → **variable sliding window** + **`map[c]` = net "need"** (positive: still need, negative: have extra) + `cnt = remaining chars to match`. **`--map[s[r]] >= 0` ⇒ cnt--**; **`++map[s[l]] > 0` ⇒ cnt++**. Shrink while `cnt == 0`, record min.
>
> **中文**: **s 中最短包含 t 全部字符 (多集) 的子串** → **不定长滑窗** + **`map[c]` = 净需求量** (正: 还缺, 负: 多余) + `cnt = 还差几个字符`. **右扩减一, ≥ 0 表示"填了必需" → cnt--**; **左缩加一, > 0 表示"踢掉必需" → cnt++**. `cnt == 0` 时缩记 min.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s` 和 `t`. 找 `s` 中**最短**子串, 使其**包含 t 的所有字符** (含重复次数). 无解返 `""`.

- 例: `s = "ADOBECODEBANC", t = "ABC"` → `"BANC"` (最短含 ABC 的子串).

**中文**: s 中含 t 所有字符的最短子串.

## Key Insights

1. **🔑 灵魂 trick: 一个 map 表达 "还需要 / 多余了" 两种语义 / One map for both "need" and "extra"**

    Yang 的核心设计:

    - **初始 `map[c] = t 中 c 的频次`** — 正数 = "还需要过 c".
    - 每次 window 加 `s[r]`: `map[s[r]]--`.
        - 减完 **≥ 0** → "s[r] 曾经是必需的, 现在填了一份需求" → `cnt--`.
        - 减完 **< 0** → "过量了, s[r] 是多余的" → cnt 不变.
    - 每次 window 缩去 `s[l]`: `map[s[l]]++`.
        - 加完 **> 0** → "把必需字符踢出窗了, 现在缺" → `cnt++`.
        - 加完 **≤ 0** → "只是踢掉多余, 无影响" → cnt 不变.

    → **map[c] 的**符号**告诉你窗口"过量" 还是"缺量"**. 一个数组两种语义.

    > **"用净差 (net) 代替 having / needed 两个 map" 是滑窗类 Hard 的极简招式**. 面试写这版能秒判 senior.

2. **🔑 `cnt` = "剩余需要匹配的字符总数" / cnt = remaining chars to match**

    初始 `cnt = t.size()`. **cnt == 0 ⇔ 窗口覆盖 t 的多集**. 用**一个 int 就够**判定 — 不用比较 26 长数组.

    > **单一 int 状态维护** 比 [0567 Permutation in String](../0567-permutation-in-string/README.md) 的 "每步比 26 长数组" 快. 关键: 只关心"是否覆盖", 不关心具体每字符.

3. **🔑 滑窗结构: 外 for right 扩, 内 while cnt==0 缩 / Outer expand, inner shrink**

    ```cpp
    while (r < n) {
        // 扩: 更新 map + cnt
        if (--map[s[r]] >= 0) cnt--;

        // 缩: cnt==0 表示当前窗口合法, 贪心缩到不合法
        while (cnt == 0) {
            记录 min;
            if (++map[s[l++]] > 0) cnt++;
        }
        r++;
    }
    ```

    - **外循环 for right**: 每步扩 1.
    - **内 while**: **只在 `cnt == 0`** 时缩, 缩到不合法为止.
    - **记 min 在缩之前** (`cnt == 0` 表示当前是合法窗口).

    > **"求最短" 滑窗模板** — 跟 [0209 Min Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) 同姿势 (**满足即缩**).

4. **🔑 记录最短的巧妙 pack: `d`, `minStart` / Compact min tracking**

    ```cpp
    int d = INT_MAX, minStart = 0;
    if (r - l + 1 < d) {
        d = r - l + 1;
        minStart = l;
    }
    ```

    - **`d`** — 当前找到的最短长度.
    - **`minStart`** — 当前最短窗口的起点.
    - 最后 `s.substr(minStart, d)` 提出子串.

    > 记 `(start, length)` 而不是 `(start, end)` — 直接跟 substr API 契合.

5. **🔑 128-长数组 vs `unordered_map` / Array size 128 for ASCII**

    Yang 用 `vector<int> map(128, 0)` — **覆盖所有 ASCII**. 因为 t / s 都可能含**大写字母** (LC 题面明确). 用 26 长会越界.

    > **看清题目字符集** — 只小写 26, 含大写 52, 含数字 128. 常见的**数组大小选择**.

6. **🔑 `--map[s[r]] >= 0` 而不是 `> 0` / `>= 0` includes zero**

    减完**恰好 0** → 说明"这是最后一份需要的" → cnt--.

    减完**> 0** → 说明"减之前 map > 0, 减完仍需要更多" → cnt-- (因为**一份需求被填**).

    → **合起来**: 减完 ≥ 0 都要 cnt--. `< 0` 才是"过量".

    对称: `++map[s[l]] > 0` 意为 "加完后 map > 0, 说明这本是必需字符, 踢出后就缺" → cnt++.

    > **这两个不等号方向要背下来**. 反了 (`> 0` / `>= 0` 写颠倒) 就是 bug.

7. **🔑 复杂度 O(n + m) 时间, O(k) 空间 / Linear**

    - Time: 摊销, 每字符 add/remove 各 ≤ 1 次.
    - Space: map 大小 (k = 字符集).

8. **🔑 滑窗题的分类回顾 / Sliding window family (updated)**

    | 题 | 姿势 | 何时缩 | 记什么 |
    |---|---|---|---|
    | [0209](../0209-minimum-size-subarray-sum/README.md) | 求最短 | 满足即缩 | min |
    | [0003](../0003-longest-substring-without-repeating-characters/README.md) | 求最长 | 违反才缩 | max |
    | [0424](../0424-longest-repeating-character-replacement/README.md) | 求最长 | 违反滑一格 (if) | max, 高水位 |
    | [0567](../0567-permutation-in-string/README.md) | 定长 | 每步同步 | 存不存在 |
    | **0076 (本题)** | **求最短** | **满足 (cnt==0) 即缩** | **min + start** |

    > 5 大姿势齐全. 记牢触发条件 + 记录时机.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string minWindow(string s, string t) {
            vector<int> map(128, 0);                                 // ASCII 覆盖大小写
            int l = 0, r = 0, cnt = t.size(), d = INT_MAX, minStart = 0;
            for (char c : t) map[c]++;                               // 初始: t 频次

            while (r < (int)s.size()) {
                // 右扩: 减完 ≥ 0 表示填了一份必需 → cnt--
                if (--map[s[r]] >= 0) cnt--;

                // 缩窗: cnt==0 时是合法窗口, 贪心缩
                while (cnt == 0) {
                    if (r - l + 1 < d) {                             // 记 min
                        d = r - l + 1;
                        minStart = l;
                    }
                    // 加完 > 0 表示踢掉了必需 → cnt++
                    if (++map[s[l++]] > 0) cnt++;
                }
                r++;
            }
            return d == INT_MAX ? "" : s.substr(minStart, d);
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def minWindow(self, s: str, t: str) -> str:
            # Counter 支持缺 key 返 0, 语义化胜过 vector<int>(128)
            need = Counter(t)
            cnt = len(t)                        # 还要匹配几个字符 (含重复)
            l = 0
            best_len = float('inf')
            best_start = 0
            for r, c in enumerate(s):
                # need[c] > 0 意味着 "c 还需要过一份"
                if need[c] > 0: cnt -= 1
                need[c] -= 1                    # 无脑减, 可能变负 (多余)

                while cnt == 0:
                    if r - l + 1 < best_len:
                        best_len = r - l + 1
                        best_start = l
                    need[s[l]] += 1
                    if need[s[l]] > 0: cnt += 1
                    l += 1
            return "" if best_len == float('inf') else s[best_start:best_start + best_len]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {string} t
     * @return {string}
     */
    var minWindow = function(s, t) {
        const map = new Array(128).fill(0);
        for (const c of t) map[c.charCodeAt(0)]++;
        let l = 0, cnt = t.length, d = Infinity, minStart = 0;

        for (let r = 0; r < s.length; r++) {
            const rc = s.charCodeAt(r);
            if (--map[rc] >= 0) cnt--;
            while (cnt === 0) {
                if (r - l + 1 < d) {
                    d = r - l + 1;
                    minStart = l;
                }
                const lc = s.charCodeAt(l);
                if (++map[lc] > 0) cnt++;
                l++;
            }
        }
        return d === Infinity ? "" : s.substring(minStart, minStart + d);
    };
    ```

## Complexity

- **Time**: O(n + m) — n = |s|, m = |t|. 摊销, 每字符 ≤ 2 次操作.
- **Space**: O(k) — k = 字符集大小.

## 相关题目

- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 最短满足和 ≥ target, 母姿势
- [0003. Longest Substring Without Repeating Characters](../0003-longest-substring-without-repeating-characters/README.md) — 最长无重复
- [0424. Longest Repeating Character Replacement](../0424-longest-repeating-character-replacement/README.md) — 最长 + k 替换
- [0567. Permutation in String](../0567-permutation-in-string/README.md) — 定长滑窗 + anagram
- [0187. Repeated DNA Sequences](../../03-hash-table/0187-repeated-dna-sequences/README.md) — 定长滑窗 + 滚动哈希
- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 频次数组母题
- 0438\. Find All Anagrams in a String (待补) — 定长 anagram 找所有起点
- 0030\. Substring with Concatenation of All Words (待补) — 定长滑窗 + 单词级
- 0159\. Longest Substring with At Most Two Distinct Characters (待补)
- 0340\. Longest Substring with At Most K Distinct Characters (待补) — 广义 k 种
- 0632\. Smallest Range Covering Elements from K Lists (待补) — 多列滑窗

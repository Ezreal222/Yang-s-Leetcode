# 0003. Longest Substring Without Repeating Characters / 无重复字符的最长子串

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Sliding Window, Hash Set, Two Pointers, String · 滑动窗口, 哈希集合, 双指针, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Longest substring with all unique chars** → **variable sliding window + hash set**: `for right` extend; **`while window has s[right]`** shrink `left` (evict `s[left]`); track `max(right - left + 1)`.
>
> **中文**: **最长无重复子串** → **不定长滑窗 + hash set**: for right 扩; **while 窗口已含 s[right]** 就 left 缩 (删 s[left]); 记 max 长度.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s`. 求**最长**"每字符只出现一次" 的子串长度.

**中文**: 最长无重复子串长度.

## Key Insights

1. **🔑 不定长滑窗 + hash set 判"窗口约束" / Variable window + hash set for constraint check**

    **窗口约束**: 里面字符互不相同. 用 `unordered_set<char>` 实时维护窗口内字符, `.count()` 判"是否已在".

    **滑窗模板**:

    ```
    for right in [0, n):
        while 窗口 + s[right] 违反约束:      # 已含 s[right]
            evict s[left]; left++
        add s[right]
        update maxLen
    ```

    > **不定长滑窗** = 双指针家族三大流派之一 (跟 [0209 Min Size Subarray](../0209-minimum-size-subarray-sum/README.md) 同族).

2. **🔑 "最长" vs "最短" — 滑窗两大姿势 / Longest vs shortest: two sliding-window flavors**

    | | [0209 (最短)](../0209-minimum-size-subarray-sum/README.md) | **0003 (本题, 最长)** |
    |---|---|---|
    | 约束 | 和 ≥ target | 字符不重复 |
    | 何时缩 | **满足条件时** (贪心缩短) | **违反条件时** (缩到再次满足) |
    | 何时记 | **满足时** 记 min | **不违反时** 记 max |
    | 记的位置 | 内层 while | 外层 for 末尾 |

    > **"求最长" = 违反时缩到不违反, 每个 right 都对应一个合法 window → 记 max**. **"求最短" = 满足时贪心缩, 每次缩都可能更短 → 记 min**. 记住这两个姿势的**镜像关系**.

3. **🔑 每字符被 add / remove 各一次 → 摊销 O(n) / Amortized O(n)**

    尽管有嵌套 `while`, **每个字符最多被**:
    - `insert` 一次 (外层 for 里).
    - `erase` 一次 (内层 while 里).

    → 总操作 ≤ 2n → **摊销 O(n)**. 跟 [0209](../0209-minimum-size-subarray-sum/README.md) 同款分析.

    > **摊销 (amortized)** 是滑窗的灵魂 — 看**总工作量** 而非单次最坏.

4. **🔑 备选招: hash map<char, 最新位置> — 一次扫不缩窗 / Alt: map to last-index, single pass**

    ```cpp
    unordered_map<char, int> last;
    int left = 0, maxLen = 0;
    for (int right = 0; right < n; right++) {
        if (last.count(s[right]))
            left = max(left, last[s[right]] + 1);       // 跳过上次同字符位
        last[s[right]] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    ```

    - **不用 while 缩窗**: 直接**跳到 "上次 s[right] 之后"**, 一步到位.
    - **`max(left, ...)`** 守护: 不能让 left 后退 (若旧位置已在窗口外).
    - **常数更快**: 只扫一遍, map 只加不删.

    > **两版**都 O(n). set 版**易读**; map+index 版**更精**. 面试写哪个都行.

5. **🔑 空间 O(min(m, n)) / Space bounded by charset**

    最坏 window 装下所有 unique 字符. **有上界 = 字符集大小** (ASCII 128, 小写英文 26). 对大数据集 → O(m) 而非 O(n).

6. **🔑 滑窗三大流派回顾 / Sliding window family**

    | 模式 | 代表 | 特点 |
    |---|---|---|
    | 同向快慢 | [0027](../0027-remove-element/README.md) / [0151 v1](../0151-reverse-words-in-a-string/README.md) | slow 写, fast 读 |
    | 对撞合拢 | [0011](../0011-container-with-most-water/README.md) / [0344](../0344-reverse-string/README.md) | 两端往中 |
    | **不定长滑窗** | **本题 + [0209](../0209-minimum-size-subarray-sum/README.md)** | 右扩左缩 |

    > 双指针家族的三大姿势各有对应场景. 记牢分类.

## Solution

=== "C++"

    **v1: hash set + while 缩 (推荐首选)**

    ```cpp
    class Solution {
    public:
        int lengthOfLongestSubstring(string s) {
            unordered_set<char> window;
            int left = 0, maxLen = 0;
            for (int right = 0; right < (int)s.size(); right++) {
                while (window.count(s[right])) {                     // 违反 → 缩
                    window.erase(s[left]);
                    left++;
                }
                window.insert(s[right]);
                maxLen = max(maxLen, right - left + 1);
            }
            return maxLen;
        }
    };
    ```

    **v2: hash map<char, last_index> (一遍扫, 常数更快)**

    ```cpp
    class Solution {
    public:
        int lengthOfLongestSubstring(string s) {
            unordered_map<char, int> last;
            int left = 0, maxLen = 0;
            for (int right = 0; right < (int)s.size(); right++) {
                if (last.count(s[right]))
                    left = max(left, last[s[right]] + 1);           // 跳过上次同字符位
                last[s[right]] = right;
                maxLen = max(maxLen, right - left + 1);
            }
            return maxLen;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1 — set 版, 直译 C++
        def lengthOfLongestSubstring(self, s: str) -> int:
            window = set()
            left = max_len = 0
            for right, c in enumerate(s):
                while c in window:
                    window.remove(s[left])
                    left += 1
                window.add(c)
                # 直接 max() 内建更 Pythonic
                max_len = max(max_len, right - left + 1)
            return max_len

        # v2 — map 版, 一遍扫
        def lengthOfLongestSubstring_map(self, s: str) -> int:
            last: dict[str, int] = {}
            left = max_len = 0
            for right, c in enumerate(s):
                # dict.get(c, -1) 缺 key 返 -1 → last_pos + 1 = 0, max(0, left) = left → 无影响
                # 一步兜边界, 免 if 判
                left = max(left, last.get(c, -1) + 1)
                last[c] = right
                max_len = max(max_len, right - left + 1)
            return max_len
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var lengthOfLongestSubstring = function(s) {
        // v1: Set 版
        const window = new Set();
        let left = 0, maxLen = 0;
        for (let right = 0; right < s.length; right++) {
            while (window.has(s[right])) {
                window.delete(s[left]);
                left++;
            }
            window.add(s[right]);
            maxLen = Math.max(maxLen, right - left + 1);
        }
        return maxLen;
    };
    ```

## Complexity

- **Time**: O(n) — 摊销, 每字符被 add/remove 各 ≤ 1 次.
- **Space**: O(min(m, n)) — m = 字符集大小.

## 相关题目

- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 最短滑窗母题
- [0011. Container With Most Water](../0011-container-with-most-water/README.md) — 对撞双指针 + 贪心
- [0187. Repeated DNA Sequences](../../03-hash-table/0187-repeated-dna-sequences/README.md) — **定长**滑窗 + 滚动哈希
- [0128. Longest Consecutive Sequence](../../03-hash-table/0128-longest-consecutive-sequence/README.md) — hash set + "只从头扩"
- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 计数数组基础
- 0076\. Minimum Window Substring (待补) — 最短窗口子串, Hard, 计数 + 双指针
- [0424. Longest Repeating Character Replacement](../0424-longest-repeating-character-replacement/README.md) — 至多 k 次替换, maxCount 不减 trick
- 0904\. Fruit Into Baskets (待补) — 至多 2 种字符的最长子数组
- 0438\. Find All Anagrams in a String (待补) — 定长滑窗 + 频次
- [0567. Permutation in String](../0567-permutation-in-string/README.md) — 定长滑窗 + 频次签名
- 0713\. Subarray Product Less Than K (待补) — 滑窗 + 计数

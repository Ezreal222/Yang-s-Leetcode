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

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to find the length of the **longest substring** of `s` where **every character appears at most once**. Let me confirm a few things:"

- "**Substring** meaning contiguous, right? Not subsequence." *(yes)*
- "What's the character set — **ASCII, lowercase letters, or full Unicode**?" *(usually ASCII 128 in LC — affects space bound.)*
- "**Case sensitive**? `'A'` and `'a'` are different?" *(yes.)*
- "What about **empty string** — return 0?" *(yes.)*

### 2. Brainstorm approaches (1 min)

> "Let me think about a few options.
>
> **Approach 1 — brute force**: try every substring `s[i..j]`, check if it has all unique chars using a set. That's O(n³) — n² substrings × O(n) check. Way too slow.
>
> **Approach 2 — sliding window with a set**: this is the natural fit. Maintain a window `[left, right]` where all chars inside are unique. Extend `right` when I can; when a duplicate appears, shrink `left` until the window is valid again. Track the max window size along the way. That's **O(n)** amortized — every character enters and leaves the set at most once.
>
> **Approach 3 — sliding window with a hash map of last-seen index**: instead of shrinking one step at a time, jump `left` directly to `last[s[right]] + 1`. Same O(n) but tighter constant, only one pass.
>
> I'll go with **approach 2** first — it's the cleanest way to explain the logic. If time permits, I'll show the map optimization as a follow-up."

### 3. Sketch the algorithm before coding (1 min)

> "The sliding-window loop:
>
> - `left = 0`, `maxLen = 0`, empty set as the window.
> - For each `right` from 0 to n-1:
>   - **While** `s[right]` is already in the window, **evict** `s[left]` from the set and advance `left`.
>   - Add `s[right]` to the set.
>   - Update `maxLen = max(maxLen, right - left + 1)`.
> - Return `maxLen`.
>
> Key detail: I use `while`, not `if`, because in principle `left` may need to advance multiple positions — though for **this specific problem** at most one, since the invariant is 'window has unique chars' and we only just added the duplicate. Still, `while` is the safer general template and doesn't hurt correctness."

> "Two subtler points worth naming out loud:
>
> 1. **'longest' vs 'shortest' sliding-window are mirror opposites**: for shortest (like Min Size Subarray Sum), you shrink **while the window is still valid**, recording min inside the while. Here you shrink **while the window is invalid**, and record max **after** the while — because every `right` corresponds to at most one valid window of maximum size ending there.
> 2. **Amortized O(n)**: even with a `while` inside a `for`, each character enters the set once and leaves once, so total work is bounded by 2n."

### 4. Code while narrating (2 min)

> "Let me write it out."

```cpp
int lengthOfLongestSubstring(string s) {
    unordered_set<char> window;
    int left = 0, maxLen = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        // Shrink until s[right] is no longer in the window
        while (window.count(s[right])) {
            window.erase(s[left]);
            left++;
        }
        window.insert(s[right]);
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

> "That's about 10 lines. Nothing tricky — the loop invariant is what carries the correctness."

### 5. Trace an example (1 min)

> "Let me trace with `s = "abcabcbb"` to verify:
>
> | `right` | `s[right]` | window before | action | window after | maxLen |
> |---|---|---|---|---|---|
> | 0 | 'a' | {} | add | {a} | 1 |
> | 1 | 'b' | {a} | add | {a,b} | 2 |
> | 2 | 'c' | {a,b} | add | {a,b,c} | **3** |
> | 3 | 'a' | {a,b,c} | evict a → {b,c}, add a | {b,c,a} | 3 |
> | 4 | 'b' | {b,c,a} | evict b → {c,a}, add b | {c,a,b} | 3 |
> | 5 | 'c' | {c,a,b} | evict c → {a,b}, add c | {a,b,c} | 3 |
> | 6 | 'b' | {a,b,c} | evict a → {b,c}, evict b → {c}, add b | {c,b} | 3 |
> | 7 | 'b' | {c,b} | evict c → {b}, evict b → {}, add b | {b} | 3 |
>
> Final answer: 3, which matches — the longest unique substring is `'abc'` (or `'bca'`, or `'cab'`)."

### 6. Complexity (20s)

> "**Time O(n)** amortized — each character is inserted and erased at most once. **Space O(min(m, n))** where `m` is the character set size — for ASCII input the set holds at most 128 entries, so it's O(1) in practice."

### 7. Optimization + follow-ups (1 min)

> "One optimization worth mentioning: I can replace the set with a **hash map of char → last-seen index**. Then instead of shrinking one step at a time inside a `while`, I jump `left` directly:"

```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> last;
    int left = 0, maxLen = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        if (last.count(s[right]))
            left = max(left, last[s[right]] + 1);  // jump, but never rewind
        last[s[right]] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

> "The `max(left, ...)` guard is important — the last seen index of `s[right]` might already be **before** `left`, meaning that duplicate is already outside the window. Without the guard, `left` would rewind and count invalid characters."

> "Related problems I'd expect as follow-ups: **0424** — longest substring where we can **replace up to k characters** to make it uniform — same window shape but with a `maxCount` trick. **0076** — **shortest** window covering all characters of a target — mirror problem. Same template, different bookkeeping. And **0567 / 0438** — fixed-length window variants for permutation matching."

> "Any follow-ups you'd like me to code?"

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
- [0076. Minimum Window Substring](../0076-minimum-window-substring/README.md) — 最短窗口子串, Hard, 计数 + 双指针
- [0424. Longest Repeating Character Replacement](../0424-longest-repeating-character-replacement/README.md) — 至多 k 次替换, maxCount 不减 trick
- 0904\. Fruit Into Baskets (待补) — 至多 2 种字符的最长子数组
- 0438\. Find All Anagrams in a String (待补) — 定长滑窗 + 频次
- [0567. Permutation in String](../0567-permutation-in-string/README.md) — 定长滑窗 + 频次签名
- 0713\. Subarray Product Less Than K (待补) — 滑窗 + 计数

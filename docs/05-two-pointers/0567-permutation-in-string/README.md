# 0567. Permutation in String / 字符串的排列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Sliding Window (Fixed), Counting, Two Pointers, String · 定长滑窗, 计数, 双指针, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/permutation-in-string/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Does `s2` contain a permutation of `s1`?** ⇔ **exists a length-`n1` substring with the same char frequencies as `s1`** → **fixed-size sliding window (len = n1)**: maintain 26-count of window; slide (+ new, − old) and compare with `s1` count.
>
> **中文**: **s2 里是否含 s1 的某个排列?** ⇔ **存在长 n1 的子串, 频次跟 s1 相同** → **定长滑窗 (长 n1)**: 维护 26 计数, 滑动 (加新减老) 后跟 s1 计数比.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 `s1` 和 `s2`. 判断 `s2` 是否**包含** `s1` 的**某个排列** (即存在 s2 的子串是 s1 的排列).

**中文**: s2 里是否含 s1 的一个排列.

## Key Insights

1. **🔑 灵魂化简: 排列 = 字符频次相同 / Permutation ⇔ same freq**

    "s1 的排列" 就是**跟 s1 有相同的字符多集**. 因此:

    - 问题变成: **s2 里存不存在长 `n1` 的子串, 其字符频次跟 s1 相同**?
    - **不需要真枚举排列** (n1! 太多).

    > 跟 [0242 Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) 的化简同源: **"字符多集" = "26 频次数组"**.

2. **🔑 定长滑窗 = 每步只改一个字符 / Fixed window: only one char changes per slide**

    窗口长**固定** = n1. 从 s2[0..n1-1] 开始, 每步:

    - **右扩**: `cnt2[s2[right]]++` (新字符入).
    - **左缩**: `cnt2[s2[right - n1]]--` (老字符出).

    → **窗口整体只**动**一格**, 计数**只改两个位置**. 每步 O(1) 更新.

    > **定长滑窗**跟**不定长滑窗** (0209 / 0003) 的区别: 定长**同步涨缩**, 大小不变. 不定长**独立涨缩**, 追求最长/最短.

3. **🔑 每步比较 26 计数数组 / Compare 26 counts each step**

    Yang 用 STL:

    ```cpp
    if (equal(begin(cnt1), end(cnt1), begin(cnt2))) return true;
    ```

    - **`std::equal`** 迭代器区间比较, 全等返 true.
    - **26 项**比较 O(26) = O(1) — **实际是常量**.

    > **总时间**: n2 × 26 = O(n2). 是**线性** (26 是常量).

4. **🔑 优化: `matches` 计数 (只在真需要时用) / Optimization: track matches count**

    进阶: 用 `int matches` 记 "多少位 cnt1[i] == cnt2[i]". 每次改一个字符时**局部维护 matches**, 判定 `matches == 26` 即可 O(1).

    ```cpp
    int matches = 0;
    // 初始化: 比一次, 相同的位 matches++
    // 每次改 cnt2[c] 前后判是否跟 cnt1[c] 一致
    ```

    - **代码 30 行**, 常数快.
    - **面试问 "还能优化吗?"** 掏这个. **Yang 的 26-比较版**是最好的可读性/性能平衡.

5. **🔑 前 n1 特判 = 建初始窗口 / Bootstrap: first n1 chars**

    ```cpp
    for (int i = 0; i < n1; i++) {
        cnt1[s1[i] - 'a']++;
        cnt2[s2[i] - 'a']++;             // 同时建 s2 前 n1 的窗口计数
    }
    if (equal(...)) return true;         // 别忘立即判一次
    ```

    - **共用一个 for**, 同时初始化 cnt1 和窗口 cnt2.
    - **立即判**因为 s2 前 n1 就可能是答案.

6. **🔑 早退 `n1 > n2` / Early return**

    若 s1 比 s2 长, 不可能是子串, 直接返 false. **一行早退**省时.

7. **🔑 跟 0438 Find All Anagrams 是双胞胎 / Twin problem: 0438**

    - **0567**: 存不存在 → 找到就返 true.
    - **0438**: 找**所有**起点位置 → 收集 vector.

    **代码 99% 一样**, 只差"return true" 和 "push_back(left) + 继续". 学一得二.

8. **🔑 复杂度 O(n2) 时间, O(1) 空间 / Linear**

    - Time: O(n1) 建初始 + O((n2 - n1) × 26) 滑动比较 = O(n2).
    - Space: 两个 26 int 常量.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool checkInclusion(string s1, string s2) {
            int n1 = s1.size(), n2 = s2.size();
            if (n1 > n2) return false;                              // 早退

            int cnt1[26] = {0}, cnt2[26] = {0};
            // 建初始窗口 (前 n1)
            for (int i = 0; i < n1; i++) {
                cnt1[s1[i] - 'a']++;
                cnt2[s2[i] - 'a']++;
            }
            if (equal(begin(cnt1), end(cnt1), begin(cnt2))) return true;

            // 滑动
            for (int right = n1; right < n2; right++) {
                cnt2[s2[right] - 'a']++;                            // 右扩
                cnt2[s2[right - n1] - 'a']--;                       // 左缩
                if (equal(begin(cnt1), end(cnt1), begin(cnt2))) return true;
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def checkInclusion(self, s1: str, s2: str) -> bool:
            n1, n2 = len(s1), len(s2)
            if n1 > n2: return False
            # Counter 直接支持 == 比较 (元素多集相等)
            # 跟 C++ 手写 26-array + equal 等价
            need = Counter(s1)
            window = Counter(s2[:n1])
            if window == need: return True
            for right in range(n1, n2):
                # +1 加新字符
                window[s2[right]] += 1
                # -1 减老字符; 若归 0 必须删 (Counter == 会区分 {a: 0} 和 {}!)
                window[s2[right - n1]] -= 1
                if window[s2[right - n1]] == 0:
                    del window[s2[right - n1]]
                if window == need: return True
            return False
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s1
     * @param {string} s2
     * @return {boolean}
     */
    var checkInclusion = function(s1, s2) {
        const n1 = s1.length, n2 = s2.length;
        if (n1 > n2) return false;
        const cnt1 = new Array(26).fill(0);
        const cnt2 = new Array(26).fill(0);
        const a = 'a'.charCodeAt(0);
        for (let i = 0; i < n1; i++) {
            cnt1[s1.charCodeAt(i) - a]++;
            cnt2[s2.charCodeAt(i) - a]++;
        }
        // 手写 26-eq helper (JS 无原生 array 比较)
        const eq = (x, y) => {
            for (let i = 0; i < 26; i++) if (x[i] !== y[i]) return false;
            return true;
        };
        if (eq(cnt1, cnt2)) return true;
        for (let right = n1; right < n2; right++) {
            cnt2[s2.charCodeAt(right) - a]++;
            cnt2[s2.charCodeAt(right - n1) - a]--;
            if (eq(cnt1, cnt2)) return true;
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(n2) — n2 × 26 (26 常量).
- **Space**: O(1) — 两个 26 int.

## 相关题目

- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 频次相等母题
- [0049. Group Anagrams](../../03-hash-table/0049-group-anagrams/README.md) — 频次签名分桶
- [0003. Longest Substring Without Repeating Characters](../0003-longest-substring-without-repeating-characters/README.md) — 不定长滑窗最长
- [0424. Longest Repeating Character Replacement](../0424-longest-repeating-character-replacement/README.md) — 不定长滑窗 + maxCount
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 最短滑窗
- [0187. Repeated DNA Sequences](../../03-hash-table/0187-repeated-dna-sequences/README.md) — 定长滑窗 + 滚动哈希
- 0438\. Find All Anagrams in a String (待补) — **本题的双胞胎**, 找所有起点
- [0076. Minimum Window Substring](../0076-minimum-window-substring/README.md) — 最短窗口子串, Hard
- 0030\. Substring with Concatenation of All Words (待补) — 定长滑窗 + 单词级

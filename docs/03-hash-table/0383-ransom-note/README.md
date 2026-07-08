# 0383. Ransom Note / 赎金信

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Table, Counting Array, String · 哈希表, 计数数组, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/ransom-note/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **`ransomNote ⊆ magazine`** (multiset sense) → **26-int counter**: `+1` for note, `−1` for mag, **any position > 0 ⇒ fail** (needed more than we had).
>
> **中文**: **note 是不是 mag 的多重子集** → **26 位计数**: note 加, mag 减, **任一位 > 0** 即需求多于供给, 返 false.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断 `ransomNote` 能不能用 `magazine` 的字符**拼出** (每字符**只能用一次**). 都是小写英文.

**中文**: note 能否用 mag 的字母凑出.

## Key Insights

1. **🔑 subset ≠ equal — 0383 是 [0242](../0242-valid-anagram/README.md) 的"子集变体" / Subset flavor of the anagram family**

    对比这两题**只差一个符号**:

    | | [0242 Valid Anagram](../0242-valid-anagram/README.md) | **0383 Ransom Note** |
    |---|---|---|
    | 判 | 两串**多集相等** | note ⊆ mag |
    | +1/-1 后 | **全 == 0** ⇒ true | **全 ≤ 0** ⇒ true |
    | 长度早退 | `size 不等 → false` | `note > mag → false` |

    > 计数数组家族的**灵魂**: **`+1/-1` 抵消模式**. 判等 vs 判子集 只差"允许供给过剩" 这一条.

2. **🔑 语义: 正数 = 需求过剩 / Positive = demand > supply**

    ```
    cnt[c] 初始 0
    note 有 c → cnt[c]++    (需求 +1)
    mag  有 c → cnt[c]--    (供给 +1, 消耗需求)
    ```

    扫完:

    - `cnt[c] > 0` — note 需要的 c 比 mag 提供的多 → **无法拼出**.
    - `cnt[c] == 0` — 刚好用完.
    - `cnt[c] < 0` — mag 有富余 (无所谓).

    > **一个数组同时表达"需要多少" 和"欠多少"** — 计数数组的美感.

3. **🔑 Yang 的代码有一处 UB: `int cnt[26];` 未初始化 / Uninitialized array = UB**

    C++ 栈上数组**默认不清零**, `cnt[i]` 初值是**随机垃圾**. 只是 LeetCode 判题机常常给零填充的栈页, 侥幸能过. 修正:

    ```cpp
    int cnt[26] = {};          // C++11+, 全零填充 (推荐)
    // 或 int cnt[26] = {0};   // 兼容 C++03
    // 或 vector<int> cnt(26); // vector 保证默认 0
    ```

    > **面试写这行**若被抓要能立刻答: "C 数组不自动初始化, 得显式 `= {}`". 老手都翻过这个船.

4. **🔑 早退优化: 长度不够直接 false / Early return on length**

    `note.size() > mag.size()` 时**不可能** 拼出 (需求总量已经超过供给). **一行早退** 省时:

    ```cpp
    if (ransomNote.size() > magazine.size()) return false;
    ```

    小优化, 面试提一下加分.

5. **🔑 计数数组家族回顾 / Counting-array family**

    | 题 | 用法 |
    |---|---|
    | [0242](../0242-valid-anagram/README.md) | **判等**: +1/-1, 全 == 0 |
    | **0383 (本题)** | **判子集**: +1/-1, **全 ≤ 0** |
    | [1002](../1002-find-common-characters/README.md) | **N 词交集**: 逐位 min |
    | [0049](../0049-group-anagrams/README.md) | **签名分桶**: 排序 or 频次编码为 key |
    | [0036](../0036-valid-sudoku/README.md) | **多张 bool 判见过** |

    > "字符集固定 → 26/128 长数组" 是快 & 简 的通用武器.

6. **🔑 通用字符集 → `unordered_map<char, int>` / Larger charset**

    题目**扩到 Unicode / 任意字符** 时, 26 长数组不够, 换 map. 逻辑不变.

    > **面试 follow-up**: "若 note / mag 含 emoji 呢?" → 换 map + 用 code point 当 key.

7. **复杂度 O(n + m) 时间, O(1) 空间 / Linear time, constant space**

    - Time: 扫两串 + 扫 26 位.
    - Space: 26 位固定.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool canConstruct(string ransomNote, string magazine) {
            if (ransomNote.size() > magazine.size()) return false;      // 早退
            int cnt[26] = {};                                            // 显式清零 (原 Yang 版是 UB)
            for (char c : ransomNote) cnt[c - 'a']++;                    // 需求 +
            for (char c : magazine)   cnt[c - 'a']--;                    // 供给 -
            for (int x : cnt) if (x > 0) return false;                   // 有需求过剩 → 拼不出
            return true;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def canConstruct(self, ransomNote: str, magazine: str) -> bool:
            # Counter 差集: Counter(note) - Counter(mag) 返回"note 里比 mag 多的部分"
            # 若结果空 (无正数残留) → 能拼出
            # 等价 C++ 手写 +1/-1 但用 Counter 语义化更强
            return not (Counter(ransomNote) - Counter(magazine))

        # —— 备选: 手动计数数组 (跟 C++ 对齐教学) ——
        def canConstruct_manual(self, ransomNote: str, magazine: str) -> bool:
            if len(ransomNote) > len(magazine): return False
            cnt = [0] * 26
            a = ord('a')
            for c in ransomNote: cnt[ord(c) - a] += 1
            for c in magazine:   cnt[ord(c) - a] -= 1
            return all(x <= 0 for x in cnt)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} ransomNote
     * @param {string} magazine
     * @return {boolean}
     */
    var canConstruct = function(ransomNote, magazine) {
        if (ransomNote.length > magazine.length) return false;
        // new Array(26).fill(0) — 基本类型 fill 无共享引用坑, 安全
        const cnt = new Array(26).fill(0);
        const a = 'a'.charCodeAt(0);
        for (const c of ransomNote) cnt[c.charCodeAt(0) - a]++;
        for (const c of magazine)   cnt[c.charCodeAt(0) - a]--;
        // .some 找到一个"需求过剩" 就短路返 true; 我们要相反, 用 !some
        return !cnt.some(x => x > 0);
    };
    ```

## Complexity

- **Time**: O(n + m) — n = |note|, m = |mag|.
- **Space**: O(1) — 26 位固定.

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 判等版母题
- [1002. Find Common Characters](../1002-find-common-characters/README.md) — N 词交集 (逐位 min)
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 签名分桶
- [0036. Valid Sudoku](../0036-valid-sudoku/README.md) — bool 数组判见过
- [0349. Intersection of Two Arrays](../0349-intersection-of-two-arrays/README.md) — hash set 交集
- 1160\. Find Words That Can Be Formed by Characters (待补) — 本题的多 note 批量版
- 0387\. First Unique Character in a String (待补) — 计数数组 + 一次扫
- 0451\. Sort Characters By Frequency (待补) — 按频次排序输出

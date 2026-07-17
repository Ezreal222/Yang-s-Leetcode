# 0266. Palindrome Permutation / 回文排列

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Table, Counting Array, Palindrome · 哈希表, 计数数组, 回文
    - **Link**: [LeetCode](https://leetcode.com/problems/palindrome-permutation/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Does some permutation of `s` form a palindrome?** ⇔ **at most 1 character has an odd count** (the potential center). 26-int count + tally odds.
>
> **中文**: **s 是否存在回文排列?** ⇔ **最多 1 个字符频次是奇数** (中心字符). 26 计数 + 数奇数个数.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断字符串 `s` 是否存在某种排列**是回文**.

例: "carerac" → 是 ("carerac" 本身或 "racecar" 都是回文排列).

**中文**: 判 s 的字符能不能重排成回文.

## Key Insights

1. **🔑 灵魂洞察: 回文 ⇔ 每字符成对 (最多 1 个奇数居中) / Palindrome ⇔ pairs + at most one solo center**

    回文结构:

    ```
    偶长: aabb → abba, 每字符出现偶数次
    奇长: aabbc → abcba, 中心 c 出现奇数次, 其他成对
    ```

    → **频次里最多 1 个奇数**. 有 0 个奇数 → 偶长回文; 有 1 个 → 奇长回文; ≥ 2 个 → **不可能**.

    > **不需要真构造回文**, 只判条件. **结构性观察** 就 3 秒解出.

2. **🔑 计数数组家族 (再次) / Counting array family (again)**

    对比 [0242 Valid Anagram](../0242-valid-anagram/README.md) 家族里, 每题**用 count 数组问不同问题**:

    | 题 | 判断 |
    |---|---|
    | [0242](../0242-valid-anagram/README.md) | 两串多集**相等** (+1/-1 全 0) |
    | [0383](../0383-ransom-note/README.md) | note ⊆ mag (+1/-1 **全 ≤ 0**) |
    | [1002](../1002-find-common-characters/README.md) | N 词交集 (**逐位 min**) |
    | **0266 (本题)** | 可回文 ⇔ **奇数频次 ≤ 1** |

    > **同一数据结构** 支持多种问题, 差别只在**最后的判定条件**. 记 pattern.

3. **🔑 Yang 的 `int cnt[26];` 又忘初始化了 / Uninit array UB (again)**

    C++ **栈上数组不自动清零**. `cnt[i]` 初值随机垃圾. LC 判题机常给零页页面, 侥幸能过. 修正:

    ```cpp
    int cnt[26] = {};        // C++11+, 全零
    ```

    > 之前 [0383](../0383-ransom-note/README.md) 里也提过同款 bug. **面试写 `int cnt[26];` 会被抓 → 立刻答"应该 `= {}`"**. 这个雷记牢.

4. **🔑 早退优化: 数到 2 个奇数就返 / Early return after 2 odds**

    ```cpp
    if (x % 2) {
        if (++count > 1) return false;
    }
    ```

    大部分不合法输入**很早就被扫出 2 个奇数**. 常数级优化, 面试可提.

5. **🔑 位掩码进阶: XOR trick (更炫) / Bitmask XOR trick**

    26 字符 → 26 bits. 每字符 **XOR 到 mask 里对应位**:

    ```cpp
    int mask = 0;
    for (char c : s) mask ^= (1 << (c - 'a'));
    // 出现偶数次: 该 bit = 0. 奇数次: bit = 1.
    // 判 mask 里 1 的个数 ≤ 1: (mask & (mask - 1)) == 0
    return (mask & (mask - 1)) == 0;
    ```

    - **XOR 消偶存奇**: 一对字符 XOR 掉自己, 落单的留 1.
    - **`mask & (mask - 1) == 0`**: 判 mask 是不是**"最多 1 个 1"** (幂次为 2 或 0) 的经典 bit trick.

    > 位掩码版**代码更短 + 常数快 + O(1) 空间**. 面试进阶答案.

6. **🔑 通用字符集: `unordered_map` / General charset**

    题目扩到 Unicode → 26 长数组不够, 换 `unordered_map<char, int>`. 逻辑不变.

7. **复杂度 O(n) 时间, O(1) 空间 / Linear**

    - Time: 扫 s + 扫 26 位.
    - Space: 26 int 固定.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool canPermutePalindrome(string s) {
            int cnt[26] = {};                            // 显式清零 (原版忘了 = UB)
            for (char c : s) cnt[c - 'a']++;
            int odd = 0;
            for (int x : cnt) {
                if (x % 2) {
                    if (++odd > 1) return false;         // 早退
                }
            }
            return true;
        }
    };

    // 备选: bitmask XOR (O(1) 空间, 常数更快)
    class SolutionBitmask {
    public:
        bool canPermutePalindrome(string s) {
            int mask = 0;
            for (char c : s) mask ^= (1 << (c - 'a'));   // 消偶存奇
            return (mask & (mask - 1)) == 0;             // ≤ 1 个 1
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def canPermutePalindrome(self, s: str) -> bool:
            # Counter(s) 返 {char: freq}. .values() 拿所有次数
            # sum(v & 1 for v in ...) — 数奇数个数. v & 1 比 v % 2 快一点点 (常数)
            # 也可用 sum(v % 2 == 1 for v in ...) — 更直白
            return sum(v & 1 for v in Counter(s).values()) <= 1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {boolean}
     */
    var canPermutePalindrome = function(s) {
        // 26-int count 或 Map 都行. 这里用 Map 是因为可能含非小写字母 (JS 无 26-限)
        // 但本题 constraints 是小写, 用 Object 键 char 更快
        const cnt = {};
        for (const c of s) cnt[c] = (cnt[c] || 0) + 1;
        let odd = 0;
        // Object.values 拿所有 count, .filter 数奇数个数
        // 也可 for (const v of Object.values(cnt)) 提前 return
        for (const v of Object.values(cnt)) {
            if (v & 1 && ++odd > 1) return false;
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n) — 扫 s + 扫计数.
- **Space**: O(1) — 26 固定 (或 O(k) 若字符集扩大).

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组母题
- [0383. Ransom Note](../0383-ransom-note/README.md) — subset 判定
- [1002. Find Common Characters](../1002-find-common-characters/README.md) — N 词交集
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 频次签名分桶
- [0246. Strobogrammatic Number](../../05-two-pointers/0246-strobogrammatic-number/README.md) — 对撞判"旋转对称"
- 0409\. Longest Palindrome (待补) — 用计数**构造** 最长回文长度
- 0125\. Valid Palindrome (待补) — 对撞双指针判回文
- 0267\. Palindrome Permutation II (待补) — **生成** 所有回文排列
- 0131\. Palindrome Partitioning (待补) — 划分成回文子串

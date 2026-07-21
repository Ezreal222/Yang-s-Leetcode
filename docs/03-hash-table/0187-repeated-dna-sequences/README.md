# 0187. Repeated DNA Sequences / 重复的 DNA 序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Set, Sliding Window, Rolling Hash, Bit Manipulation · 哈希集合, 滑动窗口, 滚动哈希, 位运算
    - **Link**: [LeetCode](https://leetcode.com/problems/repeated-dna-sequences/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find all 10-char substrings that appear >1 times** → **v1**: sliding window + hash set of substrings; **v2**: **rolling hash** — encode each char in 2 bits, roll `hash = ((hash << 2) & MASK) | code[c]`, use `unordered_set<int>` (faster hash).
>
> **中文**: **找所有出现 >1 次的 10 长子串** → **v1**: 定长滑窗 + hash set 装子串; **v2**: **滚动哈希** — 每字符 2 bits, `hash = ((hash << 2) & MASK) | code[c]` 一步滑动, int set 更快.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 DNA 序列 (只含 `A/C/G/T`). 找**所有出现次数 > 1** 的**长度 10** 子串.

**中文**: 找出现次数超过 1 的 10 长子串.

## Key Insights

1. **🔑 定长滑窗 = 枚举所有 10-长子串 / Fixed-size sliding window = enumerate all 10-substrings**

    n-9 个窗口, 每个是唯一的**起点** 决定的定长子串. 数据结构问题 = **"看过没" + "看过几次"**.

    - 见过 → 是重复的.
    - **首次**见到重复 → 添加到结果 (避免同一子串出现 3+ 次时被加多次).

    > **看到"定长子串" + "重复 / 频次"** → 反应滑窗 + hash. 是常见组合.

2. **🔑 v1 (hash set of strings): 直白 / Direct approach**

    ```
    for i in [0, n-10]:
        cur = s.substr(i, 10)
        if cur in seen: res.insert(cur)
        else: seen.insert(cur)
    ```

    - **时间**: O(n × 10) — 每次 substr + hash 都是 O(10).
    - **空间**: O(n × 10) — 存所有 unique 子串.
    - 用 **两个 set** (`seen` 和 `res`) 天然去重 (子串出现 3 次时 res 也只加一次).

    > **首选写法** — 10 是常数, O(n) 实际. **面试可以直接写这个**.

3. **🔑 v2 (rolling hash + 2-bit encoding): O(n) 精细版 / Rolling hash: refined O(n)**

    观察: DNA 只 4 种字符 → **每个字符 2 bits 就能编码** (A=00, C=01, G=10, T=11). 10 字符 = **20 bits** 完全放进一个 int.

    **滚动更新**每步 O(1) (跟每次 substr O(10) 比):

    ```cpp
    hash = ((hash << 2) & MASK) | code[s[i]];
    //     ^^^^^^^^^^^^         ^^^^^^^^^^^^
    //     左移 2 位, 腾出末位   末位塞入新字符
    //     & MASK 保留低 20 位  (踢掉最左的老字符)
    ```

    - `MASK = (1 << 20) - 1 = 0xFFFFF` — 20 个 1 的掩码.
    - **左移 2 位** ≡ 老字符"往前挪一格", 最左字符被推出去.
    - **`& MASK`** 保留低 20 bits, **等价踢掉最左字符**.
    - **`| code[c]`** 塞入新字符.

    > **滚动哈希** 的核心公式记住. 是"定长窗口 + 数值指纹" 的通用技巧, 后面 Rabin-Karp 字符串匹配也用它.

4. **🔑 为啥 v2 更快: int hash vs string hash / Why v2 wins**

    | | v1 (string) | **v2 (int)** |
    |---|---|---|
    | hash 计算 | O(10) 每次 (扫字符) | **O(1)** (identity hash on int) |
    | 相等比较 | O(10) 字符对比 | O(1) int == |
    | 存储 | 10 字节/子串 | **4 字节/int** |

    > 实测 v2 通常快 3~5×. 数据量大 (n = 100K+) 时差距明显.

5. **🔑 `repeated_keys` 防止一个子串加多次 / `repeated_keys` guards against 3+ occurrences**

    v2 里 Yang 用了额外的 `repeated_keys`:

    ```cpp
    if (seen.count(hash)) {
        if (!repeated_keys.count(hash)) {         // 第一次判为重复才 push
            repeated_keys.insert(hash);
            res.push_back(s.substr(i - 9, 10));
        }
    }
    ```

    v1 用两个 set (seen + res set) 也是同一思路. **核心问题**: 若一个子串出现 3 次, 第 2 次和第 3 次都判"重复", 但只应该往结果加一次.

    > **"首次 X" 类判定 = 需要额外的 set 记录"是否已加"**. 常见套路.

6. **🔑 `i >= 9` 才开始检 / `i >= 9` guard**

    需要至少 10 个字符才能形成一个 window. Yang 在 v2 里从 `i = 0` 开始滚动 hash, 但**只有 `i >= 9`** 才开始查表 (前 9 步只是"喂" hash).

    > 这是**滚动哈希的暖机期**. 记住这个 off-by-one.

7. **复杂度 / Complexity**

    | | Time | Space |
    |---|---|---|
    | v1 | O(n × 10) = O(n) | O(n × 10) |
    | **v2** | **O(n)** | **O(n × 4 bytes)** |

    > 常数级差异, 但都是**线性**. 面试写 v1 够. Follow-up 问"能不能更快?" 掏 v2.

## Solution

=== "C++"

    **v1: hash set of substrings (推荐首选)**

    ```cpp
    class Solution {
    public:
        vector<string> findRepeatedDnaSequences(string s) {
            unordered_set<string> st, res;
            if (s.size() < 11) return {};
            for (int i = 0; i + 10 <= (int)s.size(); i++) {
                string cur = s.substr(i, 10);
                if (st.count(cur)) res.insert(cur);              // 重复 → 加结果 (res 天然去重)
                else st.insert(cur);
            }
            return vector<string>(res.begin(), res.end());
        }
    };
    ```

    **v2: rolling hash + 2-bit encoding (更快, 面试 follow-up)**

    ```cpp
    class Solution {
    public:
        vector<string> findRepeatedDnaSequences(string s) {
            if (s.size() < 10) return {};
            unordered_map<char, int> mp = {{'A', 0}, {'C', 1}, {'G', 2}, {'T', 3}};
            unordered_set<int> seen, repeated_keys;
            vector<string> res;
            int hash = 0;
            const int MASK = (1 << 20) - 1;                      // 20 位 1

            for (int i = 0; i < (int)s.size(); i++) {
                hash = ((hash << 2) & MASK) | mp[s[i]];          // 滚动: 左移 + 掩码 + 塞新
                if (i >= 9) {                                     // 暖机后开始查表
                    if (seen.count(hash)) {
                        if (!repeated_keys.count(hash)) {          // 首次判重才加
                            repeated_keys.insert(hash);
                            res.push_back(s.substr(i - 9, 10));
                        }
                    } else {
                        seen.insert(hash);
                    }
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1 — 最 Pythonic
        def findRepeatedDnaSequences(self, s: str) -> list[str]:
            # set() 交叉两遍: seen 记"看过", res 记"重复"
            # 二者都是 set, 结果自动去重 (子串出现 3 次也只加一次到 res)
            seen, res = set(), set()
            for i in range(len(s) - 9):
                cur = s[i:i + 10]           # Python 切片 O(10), 跟 C++ substr 等价
                if cur in seen: res.add(cur)
                else: seen.add(cur)
            return list(res)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string[]}
     */
    var findRepeatedDnaSequences = function(s) {
        // v1 — 两 Set 版
        const seen = new Set(), res = new Set();
        // s.substring(i, i+10) 相当于 C++ substr(i, 10). slice 也行, 二者对正数索引等价
        for (let i = 0; i + 10 <= s.length; i++) {
            const cur = s.substring(i, i + 10);
            if (seen.has(cur)) res.add(cur);
            else seen.add(cur);
        }
        // Array.from(Set) 把 Set 展开为 Array, [...res] 是等价的 spread 写法
        return Array.from(res);
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| v1 hash set of strings | O(n × 10) | O(n × 10) |
| **v2 rolling hash** | **O(n)** | O(n × 4 bytes) |

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组基础
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 哈希分桶
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set + "只从头扩"
- [0209. Minimum Size Subarray Sum](../../05-two-pointers/0209-minimum-size-subarray-sum/README.md) — 不定长滑窗
- 0438\. Find All Anagrams in a String (待补) — 定长滑窗 + 计数
- [0567. Permutation in String](../../05-two-pointers/0567-permutation-in-string/README.md) — 定长滑窗 + 频次签名
- 0028\. Find the Index of the First Occurrence in a String (待补) — 字符串匹配, Rabin-Karp 用滚动哈希
- 0076\. Minimum Window Substring (待补) — 不定长滑窗 + 计数

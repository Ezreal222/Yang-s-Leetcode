# 2744. Find Maximum Number of String Pairs / 最大字符串配对数目

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Set, String, Pairing · 哈希集合, 字符串, 配对
    - **Link**: [LeetCode](https://leetcode.com/problems/find-maximum-number-of-string-pairs/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Count pairs (i, j) with `words[i] == reverse(words[j])`** → **one-pass hash set**: for each `w`, if `reverse(w)` was seen → `count++`, else `seen.insert(w)`.
>
> **中文**: **配对数 = 有多少对 (i, j) 满足 `words[i] == reverse(words[j])`** → **一遍扫 hash set**: 每个 w 查反转在不在, 在就计数, 不在就存起来.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串数组 `words` (**互不相同**). 求满足 `words[i] == reverse(words[j])` 且 `i < j` 的配对**个数**.

**中文**: 字符串数组求"反转配对" 数. 每对只算一次.

## Key Insights

1. **🔑 又见 "compensating value" 模式 (跟 [0001](../../01-array/0001-two-sum/README.md) 同源) / Compensating value again**

    | | 0001 Two Sum | **2744 (本题)** |
    |---|---|---|
    | 每个 x 的"另一半" | `target - x` | **`reverse(x)`** |
    | 查询 | 之前存过没 | 之前存过没 |
    | 命中动作 | 返 [seen[c], i] | count++ |
    | 未命中 | seen[x] = i | seen.insert(x) |
    | 数据结构 | map (要下标) | **set (只要存在)** |

    > **模板一样**: 一遍扫, 每个元素**"查过去" 再"存自己"**. 只是"另一半" 怎么定义因题而异.

2. **🔑 为啥是 set 不是 map / Why set, not map**

    题目只要**个数**, 不要具体下标 → **不需要**记 "这个词在哪出现". `unordered_set<string>` 够用, **省一半内存**.

    > **数据结构的选型 = 未来查询要什么**. 只要 exist → set; 要 index/count → map.

3. **🔑 一遍扫的正确性: 每对被后来者计数一次 / Correctness: each pair counted once, by the later one**

    对每对 (i, j) 且 `words[i] == reverse(words[j])`:

    - 扫到 `words[i]` 时: `reverse(words[i])` 可能没 seen 过 (还没扫到 j), 直接 insert `words[i]`.
    - 扫到 `words[j]` 时: `reverse(words[j])` = `words[i]` **已在 seen** → count++.

    → **每对只被 j 计数一次**, 不重不漏. **`i < j` 天然保证**.

4. **🔑 反转字符串一行: `string(w.rbegin(), w.rend())` / One-liner reverse via reverse iterators**

    STL 优雅招式:

    ```cpp
    string rev = string(w.rbegin(), w.rend());
    ```

    - `rbegin()` / `rend()` 是**反向迭代器** — 从末位往回扫.
    - `string(first, last)` 用迭代器区间构造 → 内容就是反过来的.

    > **迭代器区间构造 + 反迭代器 = 反转**. 比手写 `for i in n-1 downto 0: rev += s[i]` 更"惯用 STL".

5. **🔑 回文自反 case 天然处理 / Palindromic self-pair auto-handled**

    题目保证 words 互不相同. 若某词 w 是**回文** (如 "aa"): `reverse(w) == w`.

    - 扫到 w 第一次: seen 里没 (`reverse(w) = w`, 但 w 也没 seen), 存进去.
    - 若又扫到 w 第二次: 这**违反 distinct**, 不会发生.

    → 回文词**单独出现不会配对** — 因为它需要"另一个同样的 w" 来配, 但题目保证 distinct. **代码天然正确**.

    > 若题目**允许重复**, 就变成"回文词的成对个数 = ⌊出现次数 / 2⌋". 那时得用 Counter.

6. **🔑 双 for 蛮力 O(n² × L) 也能过 (n ≤ 50) / Brute force works but is slower**

    LC 里 n ≤ 50, L ≤ 50 → 蛮力 O(n² × L) = 125K 次 char compare, 也能过. 但 **hash 版是"面试标答"** — 展示思维.

    > **数据规模允许 ≠ 应该写慢版**. hash 版和蛮力**同样短**, 何不选优的.

7. **复杂度 O(n × L) 时间, O(n × L) 空间 / Linear in total chars**

    - Time: n 次扫 + 每次 reverse O(L) + hash O(L).
    - Space: hash set 存 n 个词.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int maximumNumberOfStringPairs(vector<string>& words) {
            unordered_set<string> seen;
            int count = 0;
            for (auto& w : words) {
                string rev = string(w.rbegin(), w.rend());           // 反迭代器一行反转
                if (seen.count(rev)) count++;                        // 之前存过 reverse → 配对
                else seen.insert(w);                                 // 否则存自己
            }
            return count;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maximumNumberOfStringPairs(self, words: list[str]) -> int:
            seen = set()
            count = 0
            for w in words:
                # w[::-1] 是 Python 最短的反转 — 切片 step = -1
                # 等价 C++: string(w.rbegin(), w.rend())
                if w[::-1] in seen:
                    count += 1
                else:
                    seen.add(w)
            return count
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} words
     * @return {number}
     */
    var maximumNumberOfStringPairs = function(words) {
        const seen = new Set();
        let count = 0;
        for (const w of words) {
            // JS 反转字符串: split → reverse → join. Array.from(w).reverse().join('') 也行
            // 没有 Python 的 [::-1] 或 C++ 的 rbegin() 那种一步招式
            const rev = w.split('').reverse().join('');
            if (seen.has(rev)) count++;
            else seen.add(w);
        }
        return count;
    };
    ```

## Complexity

- **Time**: O(n × L) — n 词, 每次 reverse + hash 为 O(L).
- **Space**: O(n × L) — hash set 存原词.

## 相关题目

- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — "compensating value" 母题
- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 判"字符集合" 相同
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — canonical key 分桶
- [0249. Group Shifted Strings](../0249-group-shifted-strings/README.md) — 差分签名分桶
- [0349. Intersection of Two Arrays](../0349-intersection-of-two-arrays/README.md) — hash set 交集
- 0125\. Valid Palindrome (待补) — 回文判定
- 0680\. Valid Palindrome II (待补) — 允许删一字符
- 0409\. Longest Palindrome (待补) — 计数 + 回文构造

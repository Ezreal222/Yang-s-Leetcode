# 1002. Find Common Characters / 查找共用字符

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Table, Counting Array, String · 哈希表, 计数数组, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/find-common-characters/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Chars appearing in ALL words (with multiplicity)** → **26-int min-freq array**: init from word[0]'s counter, then element-wise `min` with each subsequent word's counter → expand into result.
>
> **中文**: **每个字符在所有词中出现的最少次数** (含重复) → **26 位 min-freq 数组**: 用 word[0] 初始化, 每个后续词的频次跟它逐位取 min → 按次数展开成结果.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串数组 `words`. 找出**每个字符** 在**所有字符串** 里都出现 (含**重复次数**) 的字符, 输出成字符列表.

- 例: `["bella", "label", "roller"]` → `["e", "l", "l"]` (l 在所有词里至少 2 次, e 至少 1 次).

**中文**: 求所有词共享的字符, 保留最少出现次数倍.

## Key Insights

1. **🔑 多集交集 = 频次取 min / Multiset intersection = element-wise min of counts**

    数学上: 若某字符 c 在词 i 出现 `cnt_i(c)` 次, **交集** 里 c 出现 **`min_i cnt_i(c)`** 次.

    - `min` 是"所有集合都至少有" 的量.
    - 例: c 在词 A 出现 3 次, 词 B 2 次, 词 C 5 次 → 交集里 c 出现 min(3,2,5) = 2 次.

    > **交集 = min, 并集 = max, 差集 = 减法**. 多集运算的三条基本公式.

2. **🔑 初始化用 word[0] 省一次遍历 / Init from word[0] saves one pass**

    Yang 直接**用第一个词的频次** 作 `minFreq` 起点, 再跟后续每个词的频次 min:

    ```
    minFreq = count(words[0])
    for word in words[1:]:
        minFreq = elementwise_min(minFreq, count(word))
    ```

    等价写法是 `minFreq = [INT_MAX] * 26`, 然后**每个词** 都 min. 但那样多一次遍历, 且 INT_MAX 的语义没那么直观.

    > **单位元 (identity) 选择**: 用第一个词等价"跳过第一轮"; 用 INT_MAX 是"零元" 起点. **数学上等价, 工程上前者省一步**.

3. **🔑 计数数组的经典家族 / Counting array family**

    | 题 | 用法 |
    |---|---|
    | [0242 Valid Anagram](../0242-valid-anagram/README.md) | **两词 +1/-1 抵消** |
    | [0049 Group Anagrams](../0049-group-anagrams/README.md) | **频次签名** 当哈希 key |
    | [0036 Valid Sudoku](../0036-valid-sudoku/README.md) | **bool 数组** 判见过 |
    | **1002 (本题)** | **N 词逐位取 min** |

    > 都是"字符集小 → 数组代替 hash map" 的家族. **看到"仅小写英文"** 就本能反应 26-int.

4. **🔑 结果展开: 按频次复制 / Expand: repeat by count**

    最后一步: 对 `minFreq[i]` (代表字符 'a'+i 的出现次数), 往 `res` push `minFreq[i]` 个 `'a'+i`:

    ```cpp
    for (int i = 0; i < 26; i++)
        for (int k = 0; k < minFreq[i]; k++)
            res.push_back(string(1, 'a' + i));
    ```

    - **外层 26 位** — 按字母表顺序.
    - **内层 minFreq[i] 次** — 重复次数.
    - `string(1, ch)` 是 C++ 构造"1 长字符串" 的招式 (第一参数是长度, 第二是字符).

5. **🔑 小优化: 可以就地更新 minFreq / In-place min update**

    Yang 每个 word 都新建 `freq` 数组, 然后跟 `minFreq` 逐位取 min. **也可以** 直接把新词的频次算完就地 min 到 minFreq — 但可读性略降. Yang 的版本**清晰 + 正确**, 差异只在常数.

    > **"新建临时状态 vs 就地更新"** 是编码风格选择. 差不多性能时选**清晰**.

6. **复杂度 O(N × L + 26) / Complexity**

    - **Time**: N × L (扫每个词的每个字符) + 26 × N (逐位 min) → **O(N × L)**. L = 平均长度.
    - **Space**: O(26) = O(1) 额外. 结果列表另计.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> commonChars(vector<string>& words) {
            vector<int> minFreq(26, 0);
            for (char c : words[0]) minFreq[c - 'a']++;                  // 用第一个词初始化

            for (int i = 1; i < (int)words.size(); i++) {
                vector<int> freq(26, 0);
                for (char c : words[i]) freq[c - 'a']++;
                for (int j = 0; j < 26; j++)
                    minFreq[j] = min(minFreq[j], freq[j]);               // 逐位 min
            }

            vector<string> res;
            for (int i = 0; i < 26; i++) {
                for (int k = 0; k < minFreq[i]; k++) {
                    res.push_back(string(1, 'a' + i));                   // 构造 1 长字符串
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def commonChars(self, words: list[str]) -> list[str]:
            # Counter 的 & 运算符就是"多集交集 = 逐位 min", 一行搞定 N 词交集
            # 等价 C++: 手写 26 位 min-freq 循环
            # reduce 从左到右累积交集. 初始值是第一个 Counter, 后续每个都 &=
            from functools import reduce
            common = reduce(lambda a, b: a & b, map(Counter, words))
            # Counter.elements() 按频次展开成迭代器 — 例 Counter({'l':2}) → ['l','l']
            # 直接 list() 拿到结果
            return list(common.elements())
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} words
     * @return {string[]}
     */
    var commonChars = function(words) {
        // JS 无 Counter, 手写 26 位数组. 等价 C++ 版
        const count = (w) => {
            const a = new Array(26).fill(0);
            for (const c of w) a[c.charCodeAt(0) - 97]++;
            return a;
        };
        let minFreq = count(words[0]);
        for (let i = 1; i < words.length; i++) {
            const freq = count(words[i]);
            // .map 返回新数组; 也可以就地循环 for j in 26 — 这里 map 更函数式
            minFreq = minFreq.map((v, j) => Math.min(v, freq[j]));
        }
        const res = [];
        for (let i = 0; i < 26; i++) {
            // String.fromCharCode(97 + i) 相当于 C++ (char)('a' + i)
            const ch = String.fromCharCode(97 + i);
            for (let k = 0; k < minFreq[i]; k++) res.push(ch);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(N × L) — N 词各扫一遍, L = 平均长度.
- **Space**: O(1) 额外 (26 位固定).

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组 +1/-1 抵消
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 频次签名分桶
- [0036. Valid Sudoku](../0036-valid-sudoku/README.md) — bool 数组判见过
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set 序列
- 0383\. Ransom Note (待补) — 计数数组 "subset" 判定
- 1160\. Find Words That Can Be Formed by Characters (待补) — 用"字母表" 拼词, 逐词判 subset
- 0387\. First Unique Character in a String (待补) — 计数数组 + 一次扫
- 0451\. Sort Characters By Frequency (待补) — 按频次排序

# 0139. Word Break / 单词拆分

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack, String, Hash Table · 动态规划, 完全背包, 字符串, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/word-break/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: 给字符串 `s` 和单词字典 `wordDict`. 判断能否把 `s` 拼成**若干个字典里单词** (单词可**重复使用**) 的串接.

**中文**: 判断 `s` 能否拼成 wordDict 里若干单词的拼接 (单词可重复使用).

## Key Insights

1. **完全背包的判定变体: dp[j] = bool / Boolean flavor of complete knapsack**

    跟前面三件套 ([0518](../0518-coin-change-ii/README.md) 组合 / [0377](../0377-combination-sum-iv/README.md) 排列 / [0322](../0322-coin-change/README.md) 最值) 同模板, 这次换成**能否**:

    | 完全背包变体 | dp 含义 | 转移 |
    |---|---|---|
    | 0518 计数 (组合) | 方案数 | `dp[j] += dp[j-coin]` |
    | 0377 计数 (排列) | 序列数 | `dp[j] += dp[j-n]` |
    | 0322 最值 | 最少个数 | `dp[j] = min(dp[j], dp[j-coin]+1)` |
    | **0139 判定** | **能否拼出** | **`dp[j] \|\|= dp[j-len] && match`** |

    > 同一个完全背包骨架, 算什么决定操作符. 这就是完全背包家族的统一性.

2. **🔑 外层 j / 内层 wordDict — 跟 [0377](../0377-combination-sum-iv/README.md) 同款"排列" 顺序 / Outer position, inner words = permutation order**

    `s` 是有序字符串, "abc" 跟 "bca" 不是同一个拼接 → 顺序敏感 → 外层 `j` (位置), 内层 wordDict (物品).

    > 若是问"能否选出一些单词总长 = `|s|`" 之类无顺序需求, 才是外 word / 内 j (0518 风格). 本题必须排列.

3. **状态: dp[j] = `s[0..j-1]` 能否被拼出 / dp[j] = "prefix of length j is breakable"**

    转移: 枚举"最后拼上的那个单词" word, 若 `s` 末尾 `len(word)` 字符正好 == word 且 `dp[j-len(word)] = true`, 则 `dp[j] = true`.

    $$dp[j] = \bigvee_{w \in \text{wordDict}} \big(dp[j - |w|] \land s[j - |w| : j] = w\big)$$

    > "最后拼上的单词" 就是 "DP 思维流程" 里 [最后一步思维](../topic-dp-thinking-process.md) 的具体应用.

4. **`dp[0] = true`: 空串总能"拼出" / Empty prefix is always breakable**

    递推地基. 漏写整张 dp 都 false.

5. **🔑 `s.compare(pos, n, word)` 比 `s.substr(...) == word` 快很多 / `compare` avoids allocating temp**

    `s.substr(pos, n)` 会**新建一个 string** (堆分配 + 拷贝 O(n)), 然后再比. `s.compare(pos, n, word)` 直接在原串上比较, **不分配**:

    ```cpp
    // 原写法 (Yang 当前版本)
    if (s.substr(j - len, len) == word) ...

    // 优化版 (Yang 提到的小窍门)
    if (s.compare(j - len, len, word) == 0) ...
    ```

    LC 数据规模下两者都过, 但优化版常数明显小. **比较子串时优先 `compare`** — 这是 C++ string 的隐藏 perf 工具.

6. **两种视角: "枚举单词" (v1) vs "枚举切点" (v2) / Two angles**

    | 视角 | 外层 | 内层 | 复杂度 | 适合 |
    |---|---|---|---|---|
    | v1 枚举单词 | 位置 i | wordDict | O(n × W × L) | 字典小, 单词查比对快 |
    | v2 枚举切点 | 位置 i | 切点 j ∈ [0, i) | O(n² × L) | 字典大, 哈希查 O(L) |

    > 数据规模决定选哪个. v2 把单词集合放进 `unordered_set`, 用切点 `j` 把 `s[j..i-1]` 当 key 直接查 — 字典再大也是 O(L) 查询.

## Solution

=== "C++"
    === "v1 (Yang 原版 + compare 优化)"
        ```cpp
        class Solution {
        public:
            bool wordBreak(string s, vector<string>& wordDict) {
                int n = s.size();
                vector<bool> dp(n + 1, false);
                dp[0] = true;                                      // 空串总能拼出
                for (int j = 1; j <= n; j++) {                     // 外: 位置 → 排列顺序
                    for (const string& word : wordDict) {          // 内: 词典
                        int len = word.size();
                        // 用 compare 避免 substr 分配新字符串
                        if (j >= len && dp[j - len] &&
                            s.compare(j - len, len, word) == 0) {
                            dp[j] = true;
                            break;                                 // 一个匹配就够, 提早跳出
                        }
                    }
                }
                return dp[n];
            }
        };
        ```

    === "v2: 枚举切点 + hash set"
        ```cpp
        class Solution {
        public:
            bool wordBreak(string s, vector<string>& wordDict) {
                unordered_set<string> dict(wordDict.begin(), wordDict.end());
                int n = s.size();
                vector<bool> dp(n + 1, false);                     // dp[i] = 前 i 个字符能否拆分
                dp[0] = true;                                      // 空串可拆
                for (int i = 1; i <= n; i++) {                     // 前 i 个字符
                    for (int j = 0; j < i; j++) {                  // 枚举最后一个单词的起点 j
                        // 前 j 个能拆分 且 s[j..i-1] 是单词
                        if (dp[j] && dict.count(s.substr(j, i - j))) {
                            dp[i] = true;
                            break;                                 // 找到一种拆法即可
                        }
                    }
                }
                return dp[n];
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def wordBreak(self, s: str, wordDict: list[str]) -> bool:
            n = len(s)
            # set: O(1) 单词查重. 等价 C++ unordered_set
            word_set = set(wordDict)
            dp = [False] * (n + 1)
            dp[0] = True
            # 枚举切点版 — Pythonic 切片很轻, s[i:j] 切片做 hash 查
            for j in range(1, n + 1):
                # any(...) 早返: 任一切点成立就 True; 等价 C++ 内层 break
                dp[j] = any(dp[i] and s[i:j] in word_set for i in range(j))
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {string[]} wordDict
     * @return {boolean}
     */
    var wordBreak = function(s, wordDict) {
        const n = s.length;
        // Set 比 Array 查重快, has() O(1)
        const dict = new Set(wordDict);
        const dp = new Array(n + 1).fill(false);
        dp[0] = true;
        for (let j = 1; j <= n; j++) {
            for (let i = 0; i < j; i++) {
                // slice 类似 C++ substr, 但 JS 字符串不可变, slice 是 O(j-i)
                if (dp[i] && dict.has(s.slice(i, j))) {
                    dp[j] = true;
                    break;
                }
            }
        }
        return dp[n];
    };
    ```

## Complexity

- **Time**: O(n × W × L), n=`|s|`, W=`wordDict.size`, L= avg word len (v1). v2 是 O(n² × L).
- **Space**: O(n) + 字典存储.

## Interview Walkthrough

7-step speak-out-loud script.

### 1. Clarify

> "Just to nail down: **can words be reused?** — usually yes, unlimited reuse (complete knapsack). **Dictionary size and word lengths?** — determines whether I enumerate words or enumerate cut points. **Just true/false, or the actual decomposition?** — 0139 wants boolean; the sibling [0140](待补) wants all decompositions. And **any characters outside `[a-z]`?** — probably not, but affects if I want to prefilter."

Nail down: **reuse allowed**, **just boolean**, **all lowercase**. Now DP fits.

### 2. Brainstorm

> "Three shapes:
>
> "**Brute-force recursion**: at each position, try every word as prefix, recurse on suffix. O(2ⁿ) worst — TLE.
>
> "**Recursion + memo**: memoize the answer for each starting position. That's already O(n · W · L).
>
> "**Bottom-up DP**: `dp[j]` = 'can `s[0..j)` be broken?'. Two flavors of the inner loop:
>
> - **Enumerate words**: for each end position j, try each word — is `s[j-len..j) == word` and `dp[j-len]` true?
> - **Enumerate cut points**: for each end j, try each split point i in `[0, j)` — is `dp[i]` true AND `s[i..j)` in the dictionary hash set?
>
> "Pick based on dict size. Small dict + long strings → enumerate words. Large dict → enumerate cut points + hash set. I'll code the cut-point version; it generalizes better."

**Signal you know both.** Interviewer often steers.

### 3. Sketch

> "`dp` array of size `n+1`, all false. `dp[0] = true` — empty prefix is trivially breakable. That's the base case.
>
> "For `j` from 1 to n: try every split point `i` in `[0, j)`. If `dp[i]` is true AND the substring `s[i..j)` is in the dict set, set `dp[j] = true` and break out of the inner loop — one witness is enough.
>
> "Return `dp[n]`."

### 4. Code + narrate

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    int n = s.size();
    vector<bool> dp(n + 1, false);
    dp[0] = true;
    for (int j = 1; j <= n; j++) {
        for (int i = 0; i < j; i++) {
            if (dp[i] && dict.count(s.substr(i, j - i))) {
                dp[j] = true;
                break;                  // one witness is enough
            }
        }
    }
    return dp[n];
}
```

Narrate the two tricks:

- **"`break` after the first witness"** — we only need existence; further scans waste time.
- **"`substr` allocates; if the dict is huge and hot, I'd switch to `s.compare(i, j-i, word)` on the enumerate-words variant to skip the temp string."**

### 5. Trace

`s = "leetcode"`, `dict = {"leet", "code"}`:

| j | s[0..j) | try each split i, check dp[i] && s[i..j) in dict | dp[j] |
|---|---|---|---|
| 0 | "" | base | true |
| 1 | "l" | i=0: dp[0]=T, "l" ∉ dict | false |
| 2 | "le" | i=0: "le" ∉; i=1: dp[1]=F | false |
| 3 | "lee" | all i fail | false |
| 4 | "leet" | i=0: dp[0]=T, "leet" ∈ dict ✓ | true |
| 5..7 | ... | dp[i] false or substring not in dict | false |
| 8 | "leetcode" | i=4: dp[4]=T, "code" ∈ dict ✓ | true |

Answer: `dp[8] = true`.

**Callout: `dp[0] = true` is what makes `dp[4] = true` work** — the empty prefix is the anchor for the first word.

### 6. Complexity

> "Time: outer loop n, inner loop up to n, substring hash O(L) → **O(n² · L)**. Space: `dp` O(n), dict O(sum of word lengths). If dict is small (say W ≤ 30) and words are short, the enumerate-words variant is O(n · W · L) — often faster because W is a tiny constant."

### 7. Follow-ups

- **"Return all valid decompositions"** ([0140](待补)): recursion + memo where memo stores `list<string>` per starting position. Backtrack from the DP table.
- **"What if `s` is huge, dict small"**: prefer enumerate-words with `s.compare(...)` — avoids substr allocations. Add a **Trie** if words share prefixes to prune matches early.
- **"What if dict has millions of words"**: precompute the set of allowed word lengths — inner loop only checks those lengths. Cuts most misses.
- **"Concatenated words"** ([0472](../../03-hash-table/0472-concatenated-words/README.md)): find dict words that are themselves breakable into other dict words. Reduces to Word Break per candidate, with a smaller dict (self excluded).
- **"Add wildcards or regex"**: falls apart — need trie with wildcard matching or NFA. Different problem shape.

## 相关题目

- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包最值版
- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 完全背包组合数
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 完全背包排列数, 跟本题同"外 j" 顺序
- 0140\. Word Break II (待补) — 本题进阶: 返回**所有**拼接方案 (回溯 + 记忆化)
- 0472\. Concatenated Words (待补) — 把字典里"自身能被字典里其他词拼出" 的词全找出来
- 0091\. Decode Ways (待补) — 类似"线性拆分", 但物品是 1/2 位数字
- [§10 DP 通用思维流程 — 最后一步思维](../topic-dp-thinking-process.md) — 本题转移就是"最后拼的那个单词"

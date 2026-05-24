# 0139. Word Break / 单词拆分

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Complete Knapsack, String, Hash Table · 动态规划, 完全背包, 字符串, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/word-break/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

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

## 相关题目

- [0322. Coin Change](../0322-coin-change/README.md) — 完全背包最值版
- [0518. Coin Change II](../0518-coin-change-ii/README.md) — 完全背包组合数
- [0377. Combination Sum IV](../0377-combination-sum-iv/README.md) — 完全背包排列数, 跟本题同"外 j" 顺序
- 0140\. Word Break II (待补) — 本题进阶: 返回**所有**拼接方案 (回溯 + 记忆化)
- 0472\. Concatenated Words (待补) — 把字典里"自身能被字典里其他词拼出" 的词全找出来
- 0091\. Decode Ways (待补) — 类似"线性拆分", 但物品是 1/2 位数字
- [§10 DP 通用思维流程 — 最后一步思维](../topic-dp-thinking-process.md) — 本题转移就是"最后拼的那个单词"

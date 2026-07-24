# 0472. Concatenated Words / 连接词

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Trie, DFS, DP, Word Break, Memoization · 前缀树, 深度优先, 动态规划, 字符串拼接
    - **Link**: [LeetCode](https://leetcode.com/problems/concatenated-words/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find words that are concatenations of ≥ 2 other words in list** → **v1 Trie + DFS + memo**: walk Trie, at each `isEnd` try splitting; guard `isFirst` to enforce ≥ 2 pieces. **v2 Word Break DP**: sort by length so each word only tries **shorter** dict words (auto avoids self-use).
>
> **中文**: **找由 ≥ 2 个词典中的词拼成的词** → **v1 Trie + DFS + memo**: 沿 Trie 走, 每个 isEnd 尝试切; `isFirst` 守护保证 ≥ 2 段. **v2 Word Break DP**: **按长度排序** 让每个词只用**更短** 的词拼 (自然避免自拼自).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给不重复的**词典** `words`. 找出**所有**能由**列表中 ≥ 2 个 (可重复用) 其他词** 拼成的词.

**中文**: 找能拆成 ≥ 2 个词典词的词.

## Key Insights

1. **🔑 灵魂化简: concatenated ⇔ word 能切成 ≥ 2 段, 每段都在 dict / Definition unpacked**

    - **1 段 = 自己就在 dict** — 不算 concatenated (虽然满足 dict 匹配).
    - **≥ 2 段** — 才是我们要的.
    - → **必须**保证切分次数 ≥ 1 (即 ≥ 2 段).

    > **"起码切一次"** 是这题跟 0139 Word Break 唯一的语义差别. 落到代码就是一个 boolean flag 或"用更短的" 排序.

2. **🔑 v1 Trie + DFS + memo / v1: Trie + DFS + memo**

    整体思路:

    ```
    1. 把所有 words 塞进 Trie.
    2. 对每个 word, DFS 检查能否切成 ≥ 2 个 Trie 里的词.
    ```

    DFS 细节:

    ```cpp
    bool dfs(word, start, isFirst, memo):
        if start == word.size(): return true            // 全部切完成功
        if memo[start] != -1: return memo[start]
        沿 Trie 从 root 走:
            for i = start..n-1:
                cur = cur->children[word[i] - 'a']
                if !cur: break                          // Trie 无该分支, 死路
                if cur->isEnd:                          // 一个词结束
                    if isFirst && i == n-1: continue    // 关键守护!
                    if dfs(word, i+1, false, memo):
                        return true                     // 剩余部分也能切
        return false
    ```

    - **`isFirst` 守护**: 若这是**第一段** 且**恰好用到 word 末尾**, 意味着**整词就是 dict 里一个词** — 只 1 段, 不算 concatenated → `continue` 跳过.
    - **memo[start]**: 从 start 开始的可拼性, 避免重复 DFS.

3. **🔑 v2 Word Break DP + 排序 / v2: Word Break DP + length sort**

    思路更朴素:

    ```
    1. 按长度排序 (短的先).
    2. 遍历 words, 对每个 w 用 [0139 Word Break] DP 判 "能否用 dict (只含更短的词) 拼出".
    3. 判后再把 w 加进 dict — 保证 dict 里只有更短的.
    ```

    **关键设计**: dict 里**只有严格更短** 的词 → 每个"分段" 都是**别的词** → 自动 ≥ 2 段 (因为自己不可能在 dict 里).

    ```cpp
    sort(words by length);
    unordered_set<string> dict;
    for w in words:
        if canForm(w, dict): res.push(w);
        dict.insert(w);                                  // 加进去给后面用
    ```

    canForm 是 0139 Word Break 的标准 DP:

    ```cpp
    dp[0] = true, dp[i] = OR over j (dp[j] && dict.count(w[j:i]))
    ```

    > **"排序 + 只用更短词" 是漂亮的语义 hack** — 免去了 v1 的 `isFirst` flag.

4. **🔑 两版对比 / v1 vs v2**

    | | **v1 Trie + DFS + memo** | **v2 Word Break DP** |
    |---|---|---|
    | 数据结构 | Trie | hash set |
    | 分段判定 | 沿 Trie 遇 isEnd 分 | dp[j] + substr 查 set |
    | 分段 ≥ 2 保证 | `isFirst` flag | 排序 + 只用更短 |
    | 时间 | O(N × L²) 上界 | O(N × L²) |
    | 空间 | O(所有词字符总数 × 26) | O(所有词字符总数) |
    | 代码长度 | 长 (定义 TrieNode + insert + dfs) | 短 |

    > **v1 Trie** 在**词数 N 大**时有前缀共享优势. **v2 DP** 代码短, 数据小时更快. **两个都值得会**.

5. **🔑 `isFirst` 的精妙判定 / Subtle `isFirst` guard**

    ```cpp
    if (isFirst && i == n - 1) continue;
    ```

    这行读来奇怪, 拆开:

    - **`isFirst`**: 这是**第一段**.
    - **`i == n - 1`**: 且**恰好走到 word 末尾**.
    - **合起来**: 整个 word 作**单一段** 就在 dict — 但**只 1 段**, 违反 ≥ 2 段.
    - **`continue` 而非 `return false`**: 因为**更早的 `isEnd`** 可能还有效 (更短的前缀是 dict 词).

    > **少写这一行**: 会把"自己就是 dict 词" 也当 concatenated, WA.

6. **🔑 跟 0139 Word Break / [0648 Replace Words](../0648-replace-words/README.md) 的关系 / vs family**

    | | 0139 Word Break | 0648 Replace Words | **0472 (本题)** |
    |---|---|---|---|
    | 输入 | word + dict | sentence + roots | 只有 words |
    | 输出 | 能否拆分 (bool) | 替换后的 sentence | 所有 concatenated |
    | 分段限制 | ≥ 1 段 | 最短前缀 | **≥ 2 段** |

    > **一族三题**, dict 的用法各不同. 学一得三.

7. **🔑 复杂度 O(N × L²) / Complexity**

    - N = words 数, L = 平均长度.
    - 每个 word DFS/DP 是 O(L²) (每位置尝试 L 种切分).
    - Total: O(N × L²). Trie 版常数更好.

## Solution

=== "C++"

    **v1: Trie + DFS + memo**

    ```cpp
    class Solution {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};
            bool isEnd = false;
        };
        TrieNode* root = new TrieNode();

        void insert(const string& s) {
            TrieNode* cur = root;
            for (char c : s) {
                int idx = c - 'a';
                if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                cur = cur->children[idx];
            }
            cur->isEnd = true;
        }

        bool dfs(const string& word, int start, bool isFirst, vector<int>& memo) {
            int n = word.size();
            if (start == n) return true;
            if (memo[start] != -1) return memo[start];

            TrieNode* cur = root;
            for (int i = start; i < n; i++) {
                cur = cur->children[word[i] - 'a'];
                if (!cur) break;
                if (cur->isEnd) {
                    if (isFirst && i == n - 1) continue;        // 灵魂守护: 整词 1 段 不算
                    if (dfs(word, i + 1, false, memo)) {
                        memo[start] = 1;
                        return true;
                    }
                }
            }
            memo[start] = 0;
            return false;
        }

    public:
        vector<string> findAllConcatenatedWordsInADict(vector<string>& words) {
            for (string& w : words) insert(w);
            vector<string> res;
            for (string& w : words) {
                vector<int> memo(w.size(), -1);
                if (dfs(w, 0, true, memo)) res.push_back(w);
            }
            return res;
        }
    };
    ```

    **v2: 排序 + Word Break DP**

    ```cpp
    class Solution {
        bool canForm(const string& word, unordered_set<string>& dict) {
            if (dict.empty()) return false;
            int n = word.size();
            vector<bool> dp(n + 1, false);
            dp[0] = true;
            for (int i = 1; i <= n; i++) {
                for (int j = 0; j < i; j++) {
                    if (dp[j] && dict.count(word.substr(j, i - j))) {
                        dp[i] = true;
                        break;
                    }
                }
            }
            return dp[n];
        }
    public:
        vector<string> findAllConcatenatedWordsInADict(vector<string>& words) {
            sort(words.begin(), words.end(),
                 [](const string& a, const string& b) { return a.size() < b.size(); });
            unordered_set<string> dict;
            vector<string> res;
            for (auto& w : words) {
                if (canForm(w, dict)) res.push_back(w);
                dict.insert(w);                                 // 判后插, 保证 dict 只含更短
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v2 — 最 Pythonic
        def findAllConcatenatedWordsInADict(self, words: list[str]) -> list[str]:
            words.sort(key=len)     # 短的先
            dict_ = set()
            res = []
            for w in words:
                if self.can_form(w, dict_):
                    res.append(w)
                dict_.add(w)
            return res

        def can_form(self, word: str, dict_: set[str]) -> bool:
            if not dict_: return False
            n = len(word)
            dp = [False] * (n + 1)
            dp[0] = True
            for i in range(1, n + 1):
                for j in range(i):
                    # 短路: dp[j] 假就不用 hash 查
                    if dp[j] and word[j:i] in dict_:
                        dp[i] = True
                        break
            return dp[n]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} words
     * @return {string[]}
     */
    var findAllConcatenatedWordsInADict = function(words) {
        // v2
        words.sort((a, b) => a.length - b.length);
        const dict = new Set();
        const res = [];
        const canForm = (word) => {
            if (dict.size === 0) return false;
            const n = word.length;
            const dp = new Array(n + 1).fill(false);
            dp[0] = true;
            for (let i = 1; i <= n; i++) {
                for (let j = 0; j < i; j++) {
                    if (dp[j] && dict.has(word.substring(j, i))) {
                        dp[i] = true;
                        break;
                    }
                }
            }
            return dp[n];
        };
        for (const w of words) {
            if (canForm(w)) res.push(w);
            dict.add(w);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(N × L²) — N = 词数, L = 平均长度.
- **Space**: O(N × L) — Trie / hash set.

## 相关题目

- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — Trie 母题
- [0211. Design Add and Search Words](../0211-design-add-and-search-words-data-structure/README.md) — Trie + `.` 通配 DFS
- [0212. Word Search II](../0212-word-search-ii/README.md) — Trie + 网格 DFS 剪枝
- [0648. Replace Words](../0648-replace-words/README.md) — Trie 最短前缀替换
- 0139\. Word Break (待补) — 单词能否拆分, DP 母题
- 0140\. Word Break II (待补) — 返回所有拆分方案, DFS + memo
- 0642\. Design Search Autocomplete System (待补) — Trie + top-k
- 0820\. Short Encoding of Words (待补) — 反向 Trie

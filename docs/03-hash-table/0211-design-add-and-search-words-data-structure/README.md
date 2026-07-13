# 0211. Design Add and Search Words Data Structure / 添加与搜索单词 - 数据结构设计

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Trie, DFS, Design, Wildcard Match · 前缀树, 深度优先, 设计, 通配符
    - **Link**: [LeetCode](https://leetcode.com/problems/design-add-and-search-words-data-structure/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Trie + wildcard `.` search** → `addWord` = plain Trie insert (same as [0208](../0208-implement-trie-prefix-tree/README.md)); `search` = **DFS**: on `.`, try all 26 children (any success → true); on letter, walk one child.
>
> **中文**: **Trie + `.` 通配搜索** → `addWord` 同 0208 插入; `search` 用 **DFS**: 遇 `.` 26 个 child 分别递归 (任一成功即真); 遇字母走对应 child.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 设计支持:

- `addWord(word)` — 加词.
- `search(word)` — 词可含 `.` (匹配**任意小写字母 1 个**). 判"是否有匹配的存过的词".

**中文**: 加词 + 通配 `.` 查词.

## Key Insights

1. **🔑 `addWord` 完全跟 [0208 Trie](../0208-implement-trie-prefix-tree/README.md) 一样 / addWord = plain Trie insert**

    Trie 结构不变: `TrieNode { children[26], isEnd }`. 插入沿字符走, 缺就建, 末位打 isEnd.

    > **搭好 Trie, 变化的只是 search 逻辑**. 用同一个数据结构支持多种查询 = 设计题的美感.

2. **🔑 `.` 通配 = 分支枚举 → 必须 DFS 而不是走单路径 / Wildcard = branch → DFS, not iterate**

    普通 Trie search 沿单条路径走, 因为**每个字符只对应一个 child**. `.` 一下变了游戏:

    - 遇 `.` → 需要试**所有 26 个 child** — 只要**任一** child 分支能匹配剩余 word, 就返 true.
    - **迭代**做不到"回头再试" — 必须递归 (或显式栈 + 状态).

    > **"多分支尝试" = 反射到 DFS / 回溯**. 通配 / 正则匹配一族 (0044 / 0010) 都是这个思路.

3. **🔑 DFS 状态: `(node, i)` — 当前 Trie 位置 + word 位置 / DFS state**

    ```cpp
    bool dfs(TrieNode* node, const string& word, int i) {
        if (!node) return false;                        // 上层传 null → 死路
        if (i == word.size()) return node->isEnd;       // 词走完了 → 看是否完整词
        char c = word[i];
        if (c == '.') { ... 26 分支尝试 ... }
        else { ... 单 child 走 ... }
    }
    ```

    - **状态是 (Trie 节点, word 索引)** — 两个维度同步推进.
    - **base case: `i == word.size()`** → 词走完了, 返 `isEnd` (是完整词才算命中).
    - **兜底 `!node`** → 上层递归时可能传 null (无该 child), 直接返 false.

4. **🔑 遇 `.` 的 26 分支 / Branching on `.`**

    ```cpp
    for (int j = 0; j < 26; j++) {
        if (node->children[j] && dfs(node->children[j], word, i + 1))
            return true;
    }
    return false;
    ```

    - **`&&` 短路**: `child == null` 时不递归 (剪枝).
    - **任一 child 成功即返 true**: OR 语义.
    - 全 26 试完都失败: 返 false.

    > **短路 + OR** 是"存在性搜索" 的标准 pattern.

5. **🔑 遇字母 = 单路径走 / Letter = single-path descend**

    ```cpp
    int idx = c - 'a';
    return dfs(node->children[idx], word, i + 1);
    ```

    直接递归到对应 child. 若 child 是 null, 下一层 base case `!node` 兜底返 false.

    > **省一层 if 判**: 用 `!node` 兜底比"先判 child 存在再递归" 干净.

6. **🔑 复杂度: 最坏 O(26^k × L) / Worst case exponential in wildcard count**

    - **`k` = word 中 `.` 的个数**, L = word 长度.
    - 每个 `.` 分 26 分支 → **最坏 26^k** 条路径.
    - `word = "...."` 全通配 → **暴力遍历 Trie 中所有 L 长路径**.

    **实际**远小于最坏: Trie 稀疏, 大部分 `.` 位置的分支不到 26 个.

    > **面试提"最坏 O(26^L)"** 显示分析清晰. 实际用 Trie 剪枝很多.

7. **🔑 备选思路 / Alternatives**

    | 方案 | Time (每次 search) | Space | 备注 |
    |---|---|---|---|
    | **Trie + DFS** (Yang) | 最坏 O(26^k × L) | O(总字符 × 26) | 标答 |
    | Hash set of strings + regex match | O(N × L) | O(总字符) | 词多时慢 |
    | Hash map<length, set<string>> | O(N/K × L) | 同上 | 按长度分桶稍快 |

    > **词多 & 通配少 → Trie**. **词少 & 通配多 → 简单 hash 反而更好**.

8. **🔑 跟 [0208](../0208-implement-trie-prefix-tree/README.md) 的关系: search 从"走" 到"搜" / search: from walk to DFS**

    | | 0208 search | **0211 search** |
    |---|---|---|
    | word 里字符 | 全字母 | **可含 `.` 通配** |
    | 逻辑 | 迭代沿单路径走 | **递归 DFS** |
    | 时间 | O(L) | 最坏 O(26^k × L) |

    > **"从确定性到分支性" 的算法升级** = 迭代 → 递归. 记住这条通用规律.

## Solution

=== "C++"
    ```cpp
    class WordDictionary {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};
            bool isEnd = false;
        };
        TrieNode* root;
    public:
        WordDictionary() { root = new TrieNode(); }

        void addWord(string word) {                                  // 同 0208 insert
            TrieNode* cur = root;
            for (char c : word) {
                int idx = c - 'a';
                if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                cur = cur->children[idx];
            }
            cur->isEnd = true;
        }

        bool search(string word) { return dfs(root, word, 0); }

    private:
        bool dfs(TrieNode* node, const string& word, int i) {
            if (!node) return false;                                 // 兜底
            if (i == (int)word.size()) return node->isEnd;           // 词走完 → 完整词?

            char c = word[i];
            if (c == '.') {
                for (int j = 0; j < 26; j++) {
                    if (node->children[j] && dfs(node->children[j], word, i + 1))
                        return true;                                 // 任一分支成功即返
                }
                return false;
            } else {
                int idx = c - 'a';
                return dfs(node->children[idx], word, i + 1);        // 单路径
            }
        }
    };
    ```

=== "Python"
    ```python
    class TrieNode:
        __slots__ = ('children', 'is_end')
        def __init__(self):
            # dict 而不是 26-list — 稀疏时省内存
            self.children: dict[str, 'TrieNode'] = {}
            self.is_end = False

    class WordDictionary:
        def __init__(self):
            self.root = TrieNode()

        def addWord(self, word: str) -> None:
            cur = self.root
            for c in word:
                cur = cur.children.setdefault(c, TrieNode())
            cur.is_end = True

        def search(self, word: str) -> bool:
            def dfs(node, i):
                if node is None: return False
                if i == len(word): return node.is_end
                c = word[i]
                if c == '.':
                    # dict.values() 而不是遍历 26 个位置 — 稀疏结构下更快
                    # any(...) 短路: 任一分支返 True 就 return
                    return any(dfs(child, i + 1) for child in node.children.values())
                return dfs(node.children.get(c), i + 1)     # get() 缺 key 返 None
            return dfs(self.root, 0)
    ```

=== "JavaScript"
    ```javascript
    class TrieNode {
        constructor() {
            this.children = {};      // Object 当 dict
            this.isEnd = false;
        }
    }

    var WordDictionary = function() {
        this.root = new TrieNode();
    };

    /**
     * @param {string} word
     * @return {void}
     */
    WordDictionary.prototype.addWord = function(word) {
        let cur = this.root;
        for (const c of word) {
            if (!cur.children[c]) cur.children[c] = new TrieNode();
            cur = cur.children[c];
        }
        cur.isEnd = true;
    };

    /**
     * @param {string} word
     * @return {boolean}
     */
    WordDictionary.prototype.search = function(word) {
        const dfs = (node, i) => {
            if (!node) return false;
            if (i === word.length) return node.isEnd;
            const c = word[i];
            if (c === '.') {
                // Object.values(o) 拿所有 child. .some 短路: 任一 true 就返
                return Object.values(node.children).some(child => dfs(child, i + 1));
            }
            return dfs(node.children[c], i + 1);
        };
        return dfs(this.root, 0);
    };
    ```

## Complexity

- **`addWord`**: O(\|word\|).
- **`search`** (无 `.`): O(\|word\|).
- **`search`** (含 k 个 `.`): 最坏 O(26^k × \|word\|), 实际因 Trie 稀疏远小于.

## 相关题目

- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — Trie 母题
- 0212\. Word Search II (待补) — Trie + DFS 剪枝, LC 经典
- 0648\. Replace Words (待补) — 前缀替换
- 0677\. Map Sum Pairs (待补) — Trie + 前缀和
- 0421\. Maximum XOR of Two Numbers in an Array (待补) — 01-Trie
- 0720\. Longest Word in Dictionary (待补) — Trie / 排序 + hash
- 0044\. Wildcard Matching (待补) — `*` + `?` 通配符 DP
- 0010\. Regular Expression Matching (待补) — 正则 DP

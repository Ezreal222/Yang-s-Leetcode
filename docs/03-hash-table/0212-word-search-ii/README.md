# 0212. Word Search II / 单词搜索 II

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Trie, DFS, Backtracking, Grid, Pruning · 前缀树, 深度优先, 回溯, 网格, 剪枝
    - **Link**: [LeetCode](https://leetcode.com/problems/word-search-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find all `words` present in the grid** → **build Trie from words**, then **DFS from every cell**, walking Trie in parallel. Two killer tricks: **store `word` on TrieNode's end (`word = ""` after collecting to dedupe)**, and **`board[i][j] = '#'` mark-and-restore** for in-place visited.
>
> **中文**: **在网格里找 words 中出现的所有词** → **words 建 Trie**, 从每格 DFS 同时沿 Trie 走. 两大神招: **词存在 TrieNode 末端 (`word = ""` 防重复)**, **原地 `#` 标记 + 恢复** 免额外 visited.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符网格 `board` (m × n) 和词典 `words`. 找出 words 中**所有能在 board 里"连字符" 拼出** 的词 (每步走上/下/左/右, **一格只能用一次**).

**中文**: 网格里 4 方向走单元格拼词, 找 words 里所有能拼出的词.

## Key Insights

1. **🔑 朴素思路 O(N × m × n × 4^L) → TLE / Naive: word-by-word DFS = TLE**

    对每个 word, 从每格 DFS 试. **N 个词就 N 遍**:

    ```
    for word in words:
        for each starting cell:
            DFS looking for that word
    ```

    → **O(N × m × n × 4^L)**. words 有 10^4 个, TLE 妥妥的.

    > **"N 个模式串" 各自搜 → 一定慢**. 想合并搜索, 需要**共享前缀** — Trie 登场.

2. **🔑 灵魂反转: Trie + DFS 从**每格**出发一次 / Genius flip: DFS once per cell, walk Trie in parallel**

    换个视角:

    ```
    Build Trie from all words
    for each starting cell:
        DFS from (i, j), walking Trie in step with the grid path
        当 Trie 走到某个词末端 → 收到这个词
    ```

    **每格只 DFS 一次**, Trie 顺路帮我们"同时找所有匹配词". 时间从 `O(N × m × n × 4^L)` 降到 `O(m × n × 4^L)` — **除以 N**.

    > **"多模式共享前缀 → Trie + DFS"** 是 Aho-Corasick 的基础思想 (那个更进阶). LC 上这是 Trie 的经典应用.

3. **🔑 Trie 是**剪枝**的核心 / Trie prunes hard**

    没 Trie 的 DFS 每步都要试 4 方向, 遇 dead-end 才回溯. **有 Trie**: 每步都问"当前 char 在 Trie 里有分支吗?" 没有立刻返回. **剪枝力度大**.

    Yang 的关键一行:

    ```cpp
    if (!node->children[idx]) return;         // Trie 没这个分支 → 死路
    ```

    > **搜索题里 Trie 的价值**: **前缀不匹配时立刻剪枝**, 而不是等把整个词试完.

4. **🔑 神招 1: `string word` 存在 TrieNode 末端, 不用 `isEnd` bool / Trick 1: store word at end, not isEnd flag**

    | | 存 `bool isEnd` | **存 `string word`** (Yang) |
    |---|---|---|
    | 判"末端" | `if (isEnd) ...` | `if (!word.empty()) ...` |
    | 拿到词 | **从根回溯拼** 或**传递 word 参数** | **`word` 就是词** ✅ |
    | 内存 | 1 byte | O(词长) — 但一个 Trie 里每词只存一次 |

    → 找到匹配后**直接 `res.push_back(node->word)`**, 无需回溯 Trie 或传字符串参数. **代码短一半**.

5. **🔑 神招 2: `node->word = ""` 找到后置空 = 去重 / Trick 2: clear `word` after collecting → auto-dedupe**

    ```cpp
    if (!node->word.empty()) {
        res.push_back(node->word);
        node->word = "";              // 清空防重复
    }
    ```

    - 一个词可能被**多条路径** 拼出 (不同起点 / 走法). 不清空 → 重复添加到 res.
    - **不用额外的 `unordered_set<string>` 去重**! 直接改 Trie 的字段.

    > **"就地修改 Trie 作为去重" 是极简 hack**. 面试 subtle 加分.

6. **🔑 神招 3: `board[i][j] = '#'` 原地标记 + 恢复 = 免 visited 数组 / Trick 3: in-place `#` mark**

    ```cpp
    board[i][j] = '#';
    ... 递归 4 方向 ...
    board[i][j] = c;                  // 恢复
    ```

    - **进入前** 改成 `#` (非小写字母, DFS 里检测就返回).
    - **出去后** 恢复原字符.
    - **不用**额外 `vector<vector<bool>> visited` — 省 O(m × n) 空间.

    > **回溯题的经典就地标记**. 一进一出, "**函数结束时状态跟进入前一致**" 是回溯不变量.

7. **🔑 DFS 结构解析 / DFS structure**

    ```cpp
    void dfs(board, i, j, node, res) {
        char c = board[i][j];
        if (c == '#') return;                              // 已 visited
        int idx = c - 'a';
        if (!node->children[idx]) return;                  // Trie 无分支, 剪枝

        node = node->children[idx];                        // 走进 Trie 子节点
        if (!node->word.empty()) {                         // 命中词
            res.push_back(node->word);
            node->word = "";
        }

        board[i][j] = '#';                                 // 标记
        if (i > 0)     dfs(board, i-1, j, node, res);       // 4 方向
        if (i < m-1)   dfs(board, i+1, j, node, res);
        if (j > 0)     dfs(board, i, j-1, node, res);
        if (j < n-1)   dfs(board, i, j+1, node, res);
        board[i][j] = c;                                   // 恢复
    }
    ```

    **顺序关键**: **两个提前 return + 走 Trie + 收词 + 标记 + 递归 + 恢复**.

8. **🔑 复杂度 O(m × n × 4^L) 时间, O(K) 空间 / Complexity**

    - **Time**: 每格作起点 DFS, 最坏走 L 步 (最长词长), 每步 4 方向 (剪枝后实际远少).
    - **Space**: O(K) Trie, K = 所有 words 总字符数.

    > **`board[i][j] = '#'` 让空间 O(1)**, 不然是 O(m × n). 面试提这一句加分.

9. **🔑 可选优化: DFS 后从 Trie 里删已匹配词 / Optional: prune Trie after match**

    找到一个词后, 那条 Trie 路径**不再需要** — 若删掉, 后续 DFS 更快. 需要**回溯时清理无用子树** (从叶子往上删). 常数级优化, 面试提可加分.

## Solution

=== "C++"
    ```cpp
    class Solution {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};
            string word;                                            // 存词而非 bool isEnd
        };

        TrieNode* buildTrie(vector<string>& words) {
            TrieNode* root = new TrieNode;
            for (auto& w : words) {
                TrieNode* cur = root;
                for (char c : w) {
                    int idx = c - 'a';
                    if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                    cur = cur->children[idx];
                }
                cur->word = w;                                      // 末位存整个词
            }
            return root;
        }

    public:
        vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
            TrieNode* root = buildTrie(words);
            vector<string> res;
            int m = board.size(), n = board[0].size();
            for (int i = 0; i < m; i++)
                for (int j = 0; j < n; j++)
                    dfs(board, i, j, root, res);
            return res;
        }

    private:
        void dfs(vector<vector<char>>& board, int i, int j, TrieNode* node, vector<string>& res) {
            char c = board[i][j];
            if (c == '#') return;                                    // 已 visited
            int idx = c - 'a';
            if (!node->children[idx]) return;                        // Trie 剪枝

            node = node->children[idx];
            if (!node->word.empty()) {                               // 命中
                res.push_back(node->word);
                node->word = "";                                     // 清空防重复
            }

            board[i][j] = '#';                                       // 原地标记
            int m = board.size(), n = board[0].size();
            if (i > 0)      dfs(board, i - 1, j, node, res);
            if (i < m - 1)  dfs(board, i + 1, j, node, res);
            if (j > 0)      dfs(board, i, j - 1, node, res);
            if (j < n - 1)  dfs(board, i, j + 1, node, res);
            board[i][j] = c;                                         // 回溯恢复
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findWords(self, board: list[list[str]], words: list[str]) -> list[str]:
            # 用嵌套 dict 当 Trie — 空 key '$' 存"这里结束的完整词" (Python 惯用招)
            root = {}
            for w in words:
                node = root
                for c in w:
                    node = node.setdefault(c, {})
                node['$'] = w        # $ 是"终止 + 存词" 复用标志, 因为 $ 不是小写字母

            m, n = len(board), len(board[0])
            res = []

            def dfs(i, j, node):
                c = board[i][j]
                if c not in node: return       # Trie 无分支 或 已 visited (# 也进不了 node)
                nxt = node[c]
                # pop 一次 = 找到就删词 (天然去重, 跟 Yang C++ 的 node->word = "" 等价思想)
                if '$' in nxt:
                    res.append(nxt.pop('$'))

                board[i][j] = '#'
                for di, dj in ((-1, 0), (1, 0), (0, -1), (0, 1)):
                    ni, nj = i + di, j + dj
                    if 0 <= ni < m and 0 <= nj < n:
                        dfs(ni, nj, nxt)
                board[i][j] = c

            for i in range(m):
                for j in range(n):
                    dfs(i, j, root)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[][]} board
     * @param {string[]} words
     * @return {string[]}
     */
    var findWords = function(board, words) {
        // JS 用嵌套 Object 当 Trie. 属性 word 存"这里结束的词"
        const root = {};
        for (const w of words) {
            let node = root;
            for (const c of w) {
                if (!node[c]) node[c] = {};
                node = node[c];
            }
            node.word = w;
        }
        const m = board.length, n = board[0].length;
        const res = [];
        const dirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];

        const dfs = (i, j, node) => {
            const c = board[i][j];
            if (!(c in node)) return;       // 已 # / 无分支
            const nxt = node[c];
            if (nxt.word) {
                res.push(nxt.word);
                delete nxt.word;             // 清词防重复, 跟 C++ node->word = "" 等价
            }
            board[i][j] = '#';
            for (const [di, dj] of dirs) {
                const ni = i + di, nj = j + dj;
                if (ni >= 0 && ni < m && nj >= 0 && nj < n) dfs(ni, nj, nxt);
            }
            board[i][j] = c;
        };

        for (let i = 0; i < m; i++)
            for (let j = 0; j < n; j++)
                dfs(i, j, root);
        return res;
    };
    ```

## Complexity

- **Time**: O(m × n × 4^L) 最坏 — L = 最长词长. 实际因 Trie 剪枝**远快**.
- **Space**: O(K) Trie — K = 所有 words 总字符.

## 相关题目

- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — Trie 母题
- [0211. Design Add and Search Words](../0211-design-add-and-search-words-data-structure/README.md) — Trie + `.` 通配 DFS
- 0079\. Word Search (待补) — 单词版, 无 Trie, 纯回溯
- [0648. Replace Words](../0648-replace-words/README.md) — Trie 最短前缀替换
- 0677\. Map Sum Pairs (待补) — Trie + 前缀和
- 0720\. Longest Word in Dictionary (待补) — Trie 或 排序 + hash
- [0200. Number of Islands](../../12-graph/0200-number-of-islands/README.md) — 网格 DFS 母题
- [0130. Surrounded Regions](../../12-graph/0130-surrounded-regions/README.md) — 网格 DFS 标记 + 回溯

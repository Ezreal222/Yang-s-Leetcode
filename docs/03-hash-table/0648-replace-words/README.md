# 0648. Replace Words / 单词替换

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Trie, String, Greedy · 前缀树, 字符串, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/replace-words/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Replace each word in sentence with its shortest root prefix (if any)** → **Trie of roots**; for each word, walk Trie char-by-char, **return prefix at first `isEnd`** (shortest match); else keep original.
>
> **中文**: **每个词替换成最短的词根前缀 (若有)** → **词根建 Trie**; 每个词沿 Trie 走, **首次遇 `isEnd` 就返 prefix** (贪心最短); 无匹配保留原词.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给词根 `dictionary` + 句子 `sentence`. 把句中每个词替换成"**词典里最短的能作为它前缀的词根**"; 若无匹配保持原词. 返回替换后的句子.

**中文**: 词根替换单词, 取最短匹配前缀, 无匹配保留原词.

## Key Insights

1. **🔑 灵魂洞察: 沿 Trie 走 → 首个 isEnd 就是最短匹配前缀 / Greedy first-match**

    题目要"最短的"能替换前缀. **Trie 从根往下走, 深度递增 = 长度递增**. 所以**第一个碰到 isEnd 的节点** 就是最短匹配.

    ```
    dictionary = ["cat", "cattle", "catt"]
    word = "cattlefish"
    Trie walk: c → a → t (isEnd for "cat" 🎯 stop!) → t → l → e ...
    Return: "cat"
    ```

    > **"沿路径找最短" = 第一个成功即返**. 遇到就停. 别继续走.

2. **🔑 无匹配就保留原词 / No match ⇒ keep original**

    两种"无匹配":

    - 走到 Trie 无分支 (中途 `children[idx] == null`).
    - 词全走完仍没遇 isEnd.

    **两种都返原词**. Yang 的 v2 里 `findRoot` 两处返 word 覆盖:

    ```cpp
    if (!cur->children[idx]) return word;    // 走不下去
    ...
    return word;                             // 走完仍没 isEnd
    ```

3. **🔑 v1 vs v2 的两个关键改进 / v1 → v2 refactoring**

    Yang 自我升级了三处:

    | | v1 | v2 (improved) |
    |---|---|---|
    | Trie 存法 | `string word` (0212 style) | **`bool isEnd`** (更省内存) |
    | 分词 | 手写 `while != ' '` 拼字符 | **`stringstream >> word`** |
    | 结果拼接 | `vector<string> res` + 最后 join | **直接 string 累加**, `if (!res.empty()) res += " "` 前置空格 |
    | 无匹配 | `find` 返 "", 外层 `if-else` push | **`findRoot` 直接返 word**, 调用处一行 |

    > **v2 更 idiomatic + 短一半**. 记住这几处升级模式.

4. **🔑 `stringstream` 分词 = C++ 最干净的招式 / `stringstream` splitting**

    ```cpp
    string word;
    stringstream ss(sentence);
    while (ss >> word) {                     // 自动按空白分词
        ... 处理 word ...
    }
    ```

    - `ss >> word` **每次读一个 token** (按空白切), 到末尾返 false.
    - **代码短 + 无边界坑** (对比 v1 手写 while `sentence[i] != ' '` 需要小心 i 越界).

    > **C++ 分词首选 stringstream**. Python 有 `.split()`, JS 有 `.split(' ')` — 各语言各有一招.

5. **🔑 结果拼接: 前置空格 vs 后置分隔符 / Join with leading space**

    ```cpp
    if (!res.empty()) res += " ";     // 除第一个词, 每次先加空格
    res += findRoot(word);
    ```

    - **每次先加空格**再加词 → 结果不会有 trailing space.
    - 等价 v1 的"vector 存 + 最后 for join" 但**不需要 vector 中间**.

    > **"空 → 直接加, 非空 → 加分隔符再加"** 是通用 join 模板. 少一次特判尾部.

6. **🔑 `findRoot` 边走边拼 prefix / Build prefix while walking**

    v2 里:

    ```cpp
    string prefix;
    for (char c : word) {
        int idx = c - 'a';
        if (!cur->children[idx]) return word;
        cur = cur->children[idx];
        prefix += c;                          // 累加当前字符
        if (cur->isEnd) return prefix;        // 首个 isEnd 就返
    }
    ```

    - **必须"先走再判 isEnd"** — 因为 isEnd 描述的是"走到这个节点后是否是完整词".
    - **prefix 累加**在 "走" 的同时进行.

    > 跟 v1 存 `word` 的差异: v2 用 O(词长) 时间**边走边拼**, 换来 Trie 每节点省一个 string 字段. **时间空间 trade-off**.

7. **🔑 v1 里 `string word` 的隐藏成本 / v1's hidden cost**

    v1 每个 end 节点存 `string word`. **只有 `dictionary` 里作为完整词的节点** 存词 — 但**每个 end 节点** 都是 `sizeof(string) = 32 bytes` + 词长. Trie 总空间比 v2 (`bool isEnd = 1 byte`) 大很多.

    Trade-off: v1 找到直接返, v2 拼 prefix. **时间几乎一样, 空间 v2 胜**.

8. **复杂度 O(D + S) 时间, O(D) 空间 / Complexity**

    - D = 所有词根总字符, S = sentence 长度.
    - Build Trie: O(D). 处理句子: O(S) — 每字符走 Trie 一步.
    - Space: O(D) Trie.

## Solution

=== "C++"

    **v2 (improved): `bool isEnd` + stringstream + direct concat**

    ```cpp
    class Solution {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};
            bool isEnd = false;
        };
        TrieNode* root;

        void insert(const string& word) {
            TrieNode* cur = root;
            for (char c : word) {
                int idx = c - 'a';
                if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                cur = cur->children[idx];
            }
            cur->isEnd = true;
        }

        string findRoot(const string& word) {
            TrieNode* cur = root;
            string prefix;
            for (char c : word) {
                int idx = c - 'a';
                if (!cur->children[idx]) return word;                   // 走不下去 → 保留原词
                cur = cur->children[idx];
                prefix += c;
                if (cur->isEnd) return prefix;                          // 首个 isEnd → 最短匹配
            }
            return word;                                                // 走完没 isEnd → 保留
        }

    public:
        string replaceWords(vector<string>& dictionary, string sentence) {
            root = new TrieNode();
            for (auto& w : dictionary) insert(w);

            string res, word;
            stringstream ss(sentence);
            while (ss >> word) {
                if (!res.empty()) res += " ";                           // 前置空格
                res += findRoot(word);
            }
            return res;
        }
    };
    ```

    **v1 (original): `string word` on end nodes + manual tokenize**

    ```cpp
    class Solution {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};
            string word;                                                // 存词而非 bool
        };
        TrieNode* buildTrie(vector<string>& dictionary) {
            TrieNode* root = new TrieNode();
            for (auto& word : dictionary) {
                TrieNode* cur = root;
                for (char c : word) {
                    int idx = c - 'a';
                    if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                    cur = cur->children[idx];
                }
                cur->word = word;
            }
            return root;
        }
        string find(string word, TrieNode* root) {
            TrieNode* cur = root;
            for (char c : word) {
                int idx = c - 'a';
                if (!cur->children[idx]) return "";
                cur = cur->children[idx];
                if (!cur->word.empty()) return cur->word;               // 命中直接返
            }
            return "";
        }
    public:
        string replaceWords(vector<string>& dictionary, string sentence) {
            TrieNode* root = buildTrie(dictionary);
            vector<string> res;
            int i = 0;
            while (i < (int)sentence.size()) {
                string word;
                while (i < (int)sentence.size() && sentence[i] != ' ') word += sentence[i++];
                string rep = find(word, root);
                res.push_back(rep.empty() ? word : rep);
                i++;                                                    // 跳空格
            }
            string ans;
            for (int i = 0; i < (int)res.size() - 1; i++) ans += res[i] + ' ';
            ans += res.back();
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def replaceWords(self, dictionary: list[str], sentence: str) -> str:
            # 嵌套 dict 当 Trie. '$' 存"这里结束一个词根"标志 — 复用非小写字符防撞
            root = {}
            for w in dictionary:
                node = root
                for c in w:
                    node = node.setdefault(c, {})
                node['$'] = True                # 用 True 作 isEnd; 值本身无所谓

            def find_root(word):
                node = root
                prefix = []                     # list 累加比 string += 快 (Python str 不可变)
                for c in word:
                    if c not in node: return word
                    node = node[c]
                    prefix.append(c)
                    if '$' in node: return ''.join(prefix)      # 首个 $ 即返
                return word

            # Python .split() 默认按空白切, 一行分词. 比 C++ stringstream 更短
            # ' '.join(...) 前置分隔一步, 无 trailing
            return ' '.join(find_root(w) for w in sentence.split())
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} dictionary
     * @param {string} sentence
     * @return {string}
     */
    var replaceWords = function(dictionary, sentence) {
        const root = {};
        for (const w of dictionary) {
            let node = root;
            for (const c of w) {
                if (!node[c]) node[c] = {};
                node = node[c];
            }
            node.isEnd = true;
        }
        const findRoot = (word) => {
            let node = root;
            let prefix = '';
            for (const c of word) {
                if (!node[c]) return word;
                node = node[c];
                prefix += c;
                if (node.isEnd) return prefix;
            }
            return word;
        };
        // .split(' ') 分词, .map 处理每个词, .join(' ') 拼回句子 — 函数链一气呵成
        return sentence.split(' ').map(findRoot).join(' ');
    };
    ```

## Complexity

- **Time**: O(D + S) — D = 词根总字符, S = sentence 长度.
- **Space**: O(D) — Trie.

## 相关题目

- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — Trie 母题
- [0211. Design Add and Search Words](../0211-design-add-and-search-words-data-structure/README.md) — Trie + `.` 通配 DFS
- [0212. Word Search II](../0212-word-search-ii/README.md) — Trie + DFS 剪枝
- [0271. Encode and Decode Strings](../../04-string/0271-encode-and-decode-strings/README.md) — 字符串编码设计
- 0677\. Map Sum Pairs (待补) — Trie + 前缀和
- 0720\. Longest Word in Dictionary (待补) — Trie / 排序 + hash
- 0139\. Word Break (待补) — Trie + DP
- 0421\. Maximum XOR of Two Numbers in an Array (待补) — 01-Trie

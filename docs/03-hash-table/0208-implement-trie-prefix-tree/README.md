# 0208. Implement Trie (Prefix Tree) / 实现 Trie (前缀树)

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Trie, Design, String, Prefix Tree · 前缀树, 设计, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/implement-trie-prefix-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Trie for insert / search word / startsWith prefix** → `TrieNode { children[26], isEnd }`; walk char-by-char, create missing children on insert, mark `isEnd` at last char. **DRY**: `find(s)` shared between `search` (needs `isEnd`) and `startsWith` (just non-null).
>
> **中文**: **前缀树, 支持插入 / 查完整词 / 查前缀** → `TrieNode { children[26], isEnd }`; 一字符一字符走, 插入时缺就建, 末位打 isEnd. **DRY**: `find(s)` helper 让 search / startsWith 共用逻辑.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 实现 `Trie` 类:

- `void insert(word)` — 插入 word.
- `bool search(word)` — 是否有**完整词** word.
- `bool startsWith(prefix)` — 是否有词以 prefix 开头.

**中文**: 实现前缀树, 三个 API.

## Key Insights

1. **🔑 Trie 是"字符树" — 每边 (edge) 代表一个字符 / Trie: char-per-edge tree**

    结构:

    ```
    根 (空)
     ├─a─→ node
     │      ├─p─→ node
     │      │      ├─p─→ node (isEnd for "app")
     │      │      │      └─l─→ node
     │      │      │             └─e─→ node (isEnd for "apple")
    ```

    - **路径 = 前缀**. 从根到某节点的字符序列 = 该节点代表的前缀.
    - **`isEnd`** 标志: 这个前缀**是否恰好是一个完整词**? "app" 和 "apple" 共用前缀路径, 但两个终点都 isEnd = true.

    > **前缀共享 = Trie 的省空间根本**. N 个共享 prefix "com..." 的域名, Trie 只存一次 prefix.

2. **🔑 `children[26]` 数组 vs `unordered_map<char, TrieNode*>` / Array vs map**

    | | `children[26]` (Yang) | `unordered_map` |
    |---|---|---|
    | 字符集 | **26 固定** | 任意 (Unicode / 大字符集) |
    | 查询 | **O(1) 数组索引** | O(1) 平均 hash |
    | 内存 | **稀疏浪费** (26 × 8 字节即使空) | 只存实际用的 |
    | 常数 | **最快** | 慢一点 |

    > **26 小写英文 → 数组胜出**. 稀疏字符集 (Unicode / IP 段) → map 胜出.

3. **🔑 `insert` = "沿路径走, 缺就建" / Insert = walk + create-if-missing**

    ```cpp
    for (char c : word) {
        int idx = c - 'a';
        if (!cur->children[idx]) cur->children[idx] = new TrieNode();
        cur = cur->children[idx];
    }
    cur->isEnd = true;
    ```

    - 每步: 若这个字符**没有分支** → 新建 → 走进去. 若有 → 直接走.
    - 末位: **打 isEnd = true**. **必须**打, 不然 search 认不出这是完整词.

4. **🔑 `search` vs `startsWith` 只差一步 / search vs startsWith differ by one flag**

    Yang 抽了 helper `find(s)`:

    ```cpp
    TrieNode* find(const string& s) {
        TrieNode* cur = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!cur->children[idx]) return nullptr;         // 中途断了
            cur = cur->children[idx];
        }
        return cur;                                          // 走到尾, 返终点
    }
    ```

    - **`search(w)`**: `find(w) != nullptr && find(w)->isEnd` — 走到尾**且**是完整词.
    - **`startsWith(p)`**: `find(p) != nullptr` — 只要走到尾就行.

    > **两 API 共享 90% 逻辑, 抽 helper 是干净设计**. 跟 [0707 Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) 里 addAtHead/Tail 转 addAtIndex 同款思维.

5. **🔑 `isEnd` 缺失 = search 认成"存在" 大 bug / Missing `isEnd` = false positive**

    若忘打 isEnd, `search("app")` 就等价 `startsWith("app")` — 只要有词以 app 开头就说"app 存在". **错**.

    > **`isEnd` 是 Trie 的灵魂 flag**. 少了它, Trie 就退化成"前缀集合", 不是"词集合".

6. **🔑 复杂度 / Complexity**

    | 操作 | Time | Space (per op) |
    |---|---|---|
    | insert(w) | O(\|w\|) | 最多 \|w\| 个新节点 |
    | search(w) | O(\|w\|) | O(1) |
    | startsWith(p) | O(\|p\|) | O(1) |

    - **整体空间**: O(总字符数 × 26) — 极稀疏时**浪费** — 是 Trie 的痛点.
    - **对比 hash set of strings**: hash set 是 O(总字符数), Trie 是 O(总字符数 × 26). 但 Trie 支持**前缀查询**, hash set 不支持.

    > **"要不要前缀查询"** 是 hash set vs Trie 的分界线.

7. **🔑 进阶变体 / Advanced variants**

    - **Compressed Trie (Radix Tree / Patricia Trie)**: 单链路径压缩成一节点存整段字符串. IP 路由表用.
    - **Aho-Corasick 自动机**: 多模式串匹配, 在 Trie 上加"失败指针" (类似 KMP).
    - **持久化 Trie (Persistent Segment Tree 类)**: 版本化查询.

    > LC 上常见 0212 Word Search II (Trie + DFS 剪枝), 0648 Replace Words (前缀替换).

8. **🔑 内存管理: 析构缺失 / Missing destructor**

    Yang 的实现**没有析构** — LeetCode 不检测. 工程上需要**深度优先删所有节点**:

    ```cpp
    ~Trie() { destroy(root); }
    void destroy(TrieNode* n) {
        if (!n) return;
        for (int i = 0; i < 26; i++) destroy(n->children[i]);
        delete n;
    }
    ```

    > **面试提这一句** 加分. Python / JS 有 GC 自动清理.

## Solution

=== "C++"
    ```cpp
    class Trie {
        struct TrieNode {
            TrieNode* children[26] = {nullptr};                       // 26 分支, 默认全 null
            bool isEnd = false;
        };
        TrieNode* root;
    public:
        Trie() { root = new TrieNode(); }

        void insert(string word) {
            TrieNode* cur = root;
            for (char c : word) {
                int idx = c - 'a';
                if (!cur->children[idx]) cur->children[idx] = new TrieNode();
                cur = cur->children[idx];
            }
            cur->isEnd = true;                                        // 末位打完整词标志
        }

        bool search(string word) {
            TrieNode* node = find(word);
            return node && node->isEnd;                               // 到尾 + isEnd
        }

        bool startsWith(string prefix) {
            return find(prefix) != nullptr;                           // 只要到尾就行
        }

    private:
        // DRY: 沿 s 走到底, 中途断了返 null
        TrieNode* find(const string& s) {
            TrieNode* cur = root;
            for (char c : s) {
                int idx = c - 'a';
                if (!cur->children[idx]) return nullptr;
                cur = cur->children[idx];
            }
            return cur;
        }
    };
    ```

=== "Python"
    ```python
    class TrieNode:
        __slots__ = ('children', 'is_end')     # __slots__ 省内存 — 禁止动态属性
        def __init__(self):
            self.children: dict[str, 'TrieNode'] = {}   # dict 而不是 26-list 更 Pythonic
            self.is_end = False

    class Trie:
        def __init__(self):
            self.root = TrieNode()

        def insert(self, word: str) -> None:
            cur = self.root
            for c in word:
                # dict.setdefault(k, default): 有就返, 没就插并返 — 一步 "缺就建"
                # 跟 C++ 的 if (!ch) ch = new + walk 语义等价, 更短
                cur = cur.children.setdefault(c, TrieNode())
            cur.is_end = True

        def search(self, word: str) -> bool:
            node = self._find(word)
            return node is not None and node.is_end

        def startsWith(self, prefix: str) -> bool:
            return self._find(prefix) is not None

        def _find(self, s: str):
            cur = self.root
            for c in s:
                if c not in cur.children: return None
                cur = cur.children[c]
            return cur
    ```

=== "JavaScript"
    ```javascript
    // JS 无 nested class 语法, 用普通对象 + 工厂函数 (或 class)
    class TrieNode {
        constructor() {
            this.children = {};        // Object 当 dict, 稀疏存在的 char
            this.isEnd = false;
        }
    }

    var Trie = function() {
        this.root = new TrieNode();
    };

    /**
     * @param {string} word
     * @return {void}
     */
    Trie.prototype.insert = function(word) {
        let cur = this.root;
        for (const c of word) {
            // 兜底建: children[c] || (children[c] = new TrieNode()) — 短路 || 一步 "缺就建"
            if (!cur.children[c]) cur.children[c] = new TrieNode();
            cur = cur.children[c];
        }
        cur.isEnd = true;
    };

    Trie.prototype._find = function(s) {
        let cur = this.root;
        for (const c of s) {
            if (!cur.children[c]) return null;
            cur = cur.children[c];
        }
        return cur;
    };

    /**
     * @param {string} word
     * @return {boolean}
     */
    Trie.prototype.search = function(word) {
        const node = this._find(word);
        return node !== null && node.isEnd;
    };

    /**
     * @param {string} prefix
     * @return {boolean}
     */
    Trie.prototype.startsWith = function(prefix) {
        return this._find(prefix) !== null;
    };
    ```

## Complexity

| API | Time | Space (per op) |
|---|---|---|
| insert(w) | O(\|w\|) | O(\|w\|) 最坏 |
| search(w) | O(\|w\|) | O(1) |
| startsWith(p) | O(\|p\|) | O(1) |

**总体空间**: O(总字符数 × 字符集大小).

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 字符集处理基础
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 哈希分桶
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set + 只从头扩
- [0187. Repeated DNA Sequences](../0187-repeated-dna-sequences/README.md) — 滑窗 + 滚动哈希
- [0707. Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) — 数据结构设计, DRY 思维
- [0211. Design Add and Search Words Data Structure](../0211-design-add-and-search-words-data-structure/README.md) — Trie + `.` 通配符 (DFS 分支)
- [0212. Word Search II](../0212-word-search-ii/README.md) — Trie + DFS 剪枝 (LC 经典 Trie 应用), Hard
- 0648\. Replace Words (待补) — Trie 前缀替换
- 0677\. Map Sum Pairs (待补) — Trie + 前缀和
- 0421\. Maximum XOR of Two Numbers in an Array (待补) — 01-Trie
- 0720\. Longest Word in Dictionary (待补) — Trie / 排序 + hash

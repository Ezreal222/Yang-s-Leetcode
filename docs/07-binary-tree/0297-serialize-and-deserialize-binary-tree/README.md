# 0297. Serialize and Deserialize Binary Tree / 二叉树的序列化与反序列化

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Tree, DFS, BFS, Design, String · 二叉树, 深度优先, 广度优先, 设计, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## TL;DR / 一句话

> **EN**: **Preorder DFS with `#` sentinel for null** — writes `val, left-subtree, right-subtree` recursively. Deserialize consumes tokens in the same order; a `#` returns null immediately, halting descent. The **null markers make preorder alone reconstructible** (unlike [0105](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) which needs inorder too).
>
> **中文**: **前序 DFS + `#` 标 null** — `根, 左子树, 右子树` 递归拼串. 反序列化按同顺序消耗 token, 遇 `#` 立刻返 null. **有了 null 标记, 单前序也能唯一重建** (对比 [0105](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) 要两种遍历).
>
> *Template / 模版*: **DFS 递归 + sentinel 标 null** — 保留结构信息的通用序列化招.

## Problem

**EN**: 实现 `serialize(TreeNode*)` 和 `deserialize(string)`, 反序列化必须重建原树. 值可为任意 int (含负数).

**中文**: 序列化 + 反序列化二叉树, 互为逆.

## Key Insights

1. **🔑 灵魂: 为啥需要 null 标记 / Why null markers are essential**

    对比 [0105 Construct from Preorder + Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md): 那题需**两种遍历**才能唯一定位每个节点的左右分界. 只给 preorder `[1, 2, 3]` 有歧义 — `2` 是 `1` 的左还是右? 是 chain 还是 balanced?

    **加 `#` 标 null 之后**: preorder `[1, 2, #, #, 3, #, #]` **无歧义** — 消耗到 `#` 就知道"左子树结束". 一路递归即可.

    → **null marker 是"结构信息"的载体**. 有它, 单一遍历即够.

    > **本质**: 二叉树 = **N + 1 个空位** (N 个节点, N+1 个 null 叶挂点). 标出这 N+1 个 null 就等于标出所有结构.

2. **🔑 灵魂: 序列化 = 前序 DFS + 分隔符 / Serialize = preorder DFS + delimiter**

    ```
    dfs(node):
        if node == null: out += "#,"; return
        out += str(node.val) + ","
        dfs(node.left)
        dfs(node.right)
    ```

    - **根先写** → 反序列化时先知道父.
    - **左递归写完再写右** → 反序列化时先构造左子树, 再构造右.
    - **`,` 分隔**: 值可能多位数 (12, -5, 1000) 或负号; 无分隔无法区分.

    > **为什么用 `,` 而非空格**: 一样, 只要**不会出现在合法 token 里**都行. `,` 是最标准.

3. **🔑 灵魂: 反序列化用**流式游标 / Deserialize with a global cursor**

    ```cpp
    TreeNode* build(istringstream& in) {
        string tok;
        if (!getline(in, tok, ',')) return nullptr;
        if (tok == "#") return nullptr;
        TreeNode* node = new TreeNode(stoi(tok));
        node->left  = build(in);        // 递归先构造左, 消耗 stream 前面若干 token
        node->right = build(in);        // 剩下的自动是右子树
        return node;
    }
    ```

    **关键**: `istringstream&` **一直递给下层**, 每次 `getline(..., ',')` **消耗一个 token 并前进游标**. 左递归会把左子树用掉的 token 全消耗完, 之后 stream 头部**恰好**是右子树的第一个 token — 自动同步. 无需索引.

    > **stream 游标 = 隐式栈**. 左递归调多深, stream 就前进多远, 出栈时刚好对接右子树. 递归天然保序.

4. **🔑 `getline(in, tok, ',')` 是 C++ 拆 token 的黑话 / getline delimiter trick**

    `getline` 常被理解为"读一行", 但**第三参数是分隔符**. `getline(in, tok, ',')` = **读到下一个 `,` 为止, tok 拿到中间的**. 一次一个 token, 简洁.

    - 替代: `stringstream + >>` + 每次跳过 `,` (啰嗦).
    - 替代: 一次性 `split(',')` (O(n) 额外空间, 但下标访问更清晰).

    > **`getline(in, s, delim)` 是 C++ 拆 CSV / token 序列的标准招**. 必备.

5. **🔑 对比 BFS 层序 (LC 官方格式) / BFS-level format alternative**

    LeetCode 展示树用 `[1, 2, 3, null, null, 4, 5]` (**层序 + null padding**). 也能序列化, 但**代码更长** — 需要 queue + null 特殊处理. **DFS 递归版更短更清晰**, LC 允许.

    | 版本 | 代码量 | 空间 | 特点 |
    |---|---|---|---|
    | **DFS + `#`** | ~20 行 | O(h) 栈 | 递归天然, 面试首选 |
    | BFS 层序 + null | ~40 行 | O(w) 队 | 匹配 LC UI 格式 |

    面试问"输出格式跟 LC 一样?" → 走 BFS. 不指定 → 走 DFS.

6. **🔑 复杂度 / Complexity**

    - **Time**: O(n) 序列化 + O(n) 反序列化. 每节点常数次操作.
    - **Space**: O(n) 输出串 + O(h) 递归栈 (h = 树高, skewed 时 h = n).

7. **🔑 为啥不能只写值 (不写 null) / Why not skip null markers**

    尝试省 `#`: preorder `[1, 2, 3]`. 可能是:

    ```
        1              1               1
       /              / \               \
      2              2   3               2
     /                                  /
    3                                  3
    ```

    → **歧义**. 必须**要么加 null 标记, 要么加另一种遍历** (0105 那样).

    > **信息论**: N 个 int 值需 N 个 null marker 补齐结构信息 (总 2N+1 = 树的**外部路径长**).

8. **🔑 易错点 top 2 / Pitfalls**

    - **值有负号**: 分隔符必须避开 `-`. `,` 或空格都行, **别用 `-`**.
    - **递归共享 stream 必须传引用** `istringstream&`. 传值会**拷贝 stream 状态**, 每次都从头读 → 死循环或错. Yang 已用引用, ✅.

## Solution

=== "C++"
    ```cpp
    class Codec {
    public:
        // 序列化: 前序 DFS, null 记 "#", 分隔用 ","
        void dfs(TreeNode* node, string& out) {
            if (!node) { out += "#,"; return; }
            out += to_string(node->val) + ',';
            dfs(node->left, out);
            dfs(node->right, out);
        }

        // 反序列化: stream 游标同步消耗 token
        TreeNode* build(istringstream& in) {
            string tok;
            if (!getline(in, tok, ',')) return nullptr;      // stream 空
            if (tok == "#") return nullptr;
            TreeNode* node = new TreeNode(stoi(tok));
            node->left  = build(in);                          // 递归先耗左
            node->right = build(in);                          // 剩下是右
            return node;
        }

        string serialize(TreeNode* root) {
            string out;
            dfs(root, out);
            return out;
        }

        TreeNode* deserialize(string data) {
            istringstream in(data);
            return build(in);
        }
    };
    ```

=== "Python"
    ```python
    class Codec:
        # 分隔符用 ',', null 用 '#'. 一切跟 C++ 版逻辑一致

        def serialize(self, root: 'TreeNode | None') -> str:
            # 用 list 累积再 join, 比 str += 每步都建新 str 快得多
            # (Python str 不可变, s += x 是 O(n))
            out: list[str] = []

            def dfs(node: 'TreeNode | None') -> None:
                if not node:
                    out.append('#')
                    return
                out.append(str(node.val))
                dfs(node.left)
                dfs(node.right)

            dfs(root)
            return ','.join(out)

        def deserialize(self, data: str) -> 'TreeNode | None':
            # iter() 让 next() 逐个取; 相当于 C++ 的 stream 游标
            # 每次 next(it) 消耗一个 token, 递归天然共享游标
            it = iter(data.split(','))

            def build() -> 'TreeNode | None':
                tok = next(it)
                if tok == '#':
                    return None
                node = TreeNode(int(tok))
                node.left  = build()                          # 先耗左
                node.right = build()                          # 剩下是右
                return node

            return build()
    ```

=== "JavaScript"
    ```javascript
    // 分隔符 ',' + null 标 '#', 逻辑同 C++

    var serialize = function(root) {
        // Array push + join 比 string += 快 (JS 字符串也不可变)
        const out = [];
        const dfs = (node) => {
            if (!node) { out.push('#'); return; }
            out.push(node.val);
            dfs(node.left);
            dfs(node.right);
        };
        dfs(root);
        return out.join(',');
    };

    var deserialize = function(data) {
        // split(',') 一次性拆; 用 index 变量作游标
        // 闭包让 build() 共享 tokens + i, 相当于 C++ 的 stream&
        const tokens = data.split(',');
        let i = 0;
        const build = () => {
            const tok = tokens[i++];                          // 消耗并前进
            if (tok === '#') return null;
            const node = new TreeNode(Number(tok));
            node.left  = build();
            node.right = build();
            return node;
        };
        return build();
    };
    ```

## Complexity

| 操作 | Time | Space |
|---|---|---|
| serialize | O(n) | O(n) 输出 + O(h) 栈 |
| deserialize | O(n) | O(n) tokens + O(h) 栈 |

## Interview Walkthrough

7-step speak-out-loud script.

### 1. Clarify

> "A few questions. **Value range** — can node values be negative or multi-digit? Because that decides my delimiter. **Format** — do you want it to match LeetCode's bracket format `[1,2,3,null,null,4,5]`, or can I invent one, as long as it round-trips? And **is the tree normal binary or BST?** If it's a BST I can save space by skipping null markers, since inorder + BST invariants would reconstruct."

Nail down: **any int**, **any format ok**, **general binary tree**. Now I can pick preorder DFS + `#`.

### 2. Brainstorm

> "Two approaches. **BFS level-order with null padding** — matches LC's UI but takes ~40 lines with a queue and edge cases. **DFS preorder with `#` for null** — takes ~20 lines, purely recursive, very symmetric between serialize and deserialize. I'll go DFS unless you need the LC format specifically.
>
> "The key insight either way: **you need markers for null children**. Preorder without them isn't unique — that's why [0105](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) needs inorder too. Adding N+1 null markers turns preorder alone into a bijection with tree structure."

### 3. Sketch

Serialize:

> "Recursive DFS. If node is null, append `#,`. Else append `val,` then recurse left, then recurse right. So the string is `root, left-subtree-serialized, right-subtree-serialized`."

Deserialize:

> "Wrap the string in a stream. Read one comma-separated token: if `#`, return null. Else make a node, then `node.left = build(stream)` (which consumes exactly the left subtree's tokens), then `node.right = build(stream)`. The stream cursor stays in sync because left recursion consumes exactly what serialize wrote for the left subtree."

**Sell the stream idea:** "The stream is basically an implicit index — I don't have to pass a `pos` int by reference. Just pass the stream by reference."

### 4. Code + narrate

```cpp
class Codec {
public:
    void dfs(TreeNode* node, string& out) {
        if (!node) { out += "#,"; return; }
        out += to_string(node->val) + ',';
        dfs(node->left, out);
        dfs(node->right, out);
    }
    TreeNode* build(istringstream& in) {
        string tok;
        if (!getline(in, tok, ',')) return nullptr;
        if (tok == "#") return nullptr;
        TreeNode* node = new TreeNode(stoi(tok));
        node->left  = build(in);
        node->right = build(in);
        return node;
    }
    string serialize(TreeNode* root)  { string s; dfs(root, s); return s; }
    TreeNode* deserialize(string data){ istringstream in(data); return build(in); }
};
```

Narrate the two tricks:

- **"`getline(in, tok, ',')` with a delimiter argument is the standard C++ way to tokenize a CSV-like string one field at a time."**
- **"`istringstream&` — I pass by reference so the cursor advances globally. Passing by value would copy the stream and reset the cursor, infinite loop."**

### 5. Trace

Tree `[1, 2, 3, null, null, 4, 5]`:

```
    1
   / \
  2   3
     / \
    4   5
```

Serialize (preorder with null markers):

- visit 1 → out = `"1,"`
- visit 2 → out = `"1,2,"`
- 2.left null → `"1,2,#,"`
- 2.right null → `"1,2,#,#,"`
- visit 3 → `"1,2,#,#,3,"`
- visit 4 → `"1,2,#,#,3,4,"`
- 4.left null → `"1,2,#,#,3,4,#,"`
- 4.right null → `"1,2,#,#,3,4,#,#,"`
- visit 5 → `"1,2,#,#,3,4,#,#,5,"`
- 5.left null → `...5,#,`
- 5.right null → `...5,#,#,`

Final: `"1,2,#,#,3,4,#,#,5,#,#,"` — 11 tokens (5 nodes + 6 nulls = 2n+1 ✓).

Deserialize consumes in the same order and rebuilds. Cursor after `build` for `node=2` has consumed exactly `2, #, #` — three tokens — leaving `3, 4, #, #, 5, #, #` for `1.right`.

### 6. Complexity

> "Time O(n) both ways — each node touched once. Space O(n) for the output string plus O(h) recursion stack, where h is tree height (worst case n for skewed). Token count is 2n+1 — that's the theoretical minimum for encoding an unlabeled binary tree structure plus values."

### 7. Follow-ups

- **"Make it work for a BST"** ([0449](待补)): drop the null markers — save ~50% space. Use only preorder values; on deserialize, use a range `[lo, hi]` recursion: if next token fits `[lo, hi]`, consume and recurse `[lo, val-1]` then `[val+1, hi]`. The BST property makes structure recoverable.
- **"Values can contain commas"**: switch to a length-prefix format: `3:val,3:val,1:#,...` or JSON encode each value.
- **"Reduce the string size"**: use **layer-order without trailing nulls** — LC's format. Slightly smaller for near-complete trees, worse for skewed.
- **"Handle N-ary trees"** ([0428](待补)): serialize as `val,childCount,children...`. Child count replaces null markers.
- **"Detect duplicate subtrees"** ([0652](待补)): use the serialized string of each subtree as a hash key. Same string = same shape + values.
- **"Concurrent deserialization"**: split by top-level subtree once, spawn a worker per subtree. Preorder makes the split point cheap to find.

## 相关题目

- [0105. Construct Binary Tree from Preorder and Inorder](../0105-construct-binary-tree-from-preorder-and-inorder-traversal/README.md) — 无 null marker 版本, 需两种遍历
- [0106. Construct Binary Tree from Inorder and Postorder](../0106-construct-binary-tree-from-inorder-and-postorder-traversal/README.md) — 同上, 后序版
- [0144. Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 前序遍历母题
- [0094. Inorder Traversal](../0094-binary-tree-inorder-traversal/README.md) — 中序母题
- [0102. Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — BFS 层序 (若用 BFS 版本序列化)
- [0146. LRU Cache](../../03-hash-table/0146-lru-cache/README.md) — 另一个"多结构合作" 设计题
- 0449\. Serialize and Deserialize BST (待补) — BST 版本, 无需 null marker (BST 性质补足信息)
- 0428\. Serialize and Deserialize N-ary Tree (待补) — N-叉版本, 需带 childCount
- 0652\. Find Duplicate Subtrees (待补) — 用序列化标识子树识重
